# State Management — TanStack Query v5 + Zustand v5

## Quando usar cada um

| State | Ferramenta | Razão |
|-------|-----------|-------|
| Dados do servidor (Supabase, Airtable, fetch) | TanStack Query | Cache, deduplication, background refresh, SSR hydration |
| Dados de formulário multi-step | Zustand | Transiente, local, sem async |
| UI flags (modal aberto, sidebar, aba selecionada) | Zustand | Puro state de cliente |
| Preferências do usuário (tema, locale) | Zustand + `persist` | Persistência local |
| Notificações/toasts globais | Zustand | Event-driven, efêmero |
| Seleção de linha em tabela | Zustand | UI state derivado de ação do usuário |
| Data derivada de server data | TanStack Query `select` | Mantém derivação próxima da fonte |

---

## TanStack Query v5 — Setup Completo

### QueryClient — sem singleton no servidor

```ts
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query'

let browserClient: QueryClient | undefined

export function getQueryClient() {
  if (typeof window === 'undefined') {
    // SERVIDOR: nova instância a cada chamada (= a cada request em RSC)
    return new QueryClient({
      defaultOptions: { queries: { staleTime: 60 * 1000 } },
    })
  }
  // CLIENTE: singleton estável
  if (!browserClient) {
    browserClient = new QueryClient({
      defaultOptions: { queries: { staleTime: 60 * 1000 } },
    })
  }
  return browserClient
}
```

### Provider

```tsx
// app/providers.tsx
'use client'
import { QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { getQueryClient } from '@/lib/query-client'

export function Providers({ children }: { children: React.ReactNode }) {
  const queryClient = getQueryClient()
  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools />
    </QueryClientProvider>
  )
}
```

### Prefetch + HydrationBoundary (padrão obrigatório)

```tsx
// Server Component
import { dehydrate, HydrationBoundary } from '@tanstack/react-query'
import { getQueryClient } from '@/lib/query-client'

export default async function LeadsPage() {
  const queryClient = getQueryClient()

  // Prefetch paralelo de múltiplas queries
  await Promise.all([
    queryClient.prefetchQuery({ queryKey: ['leads'], queryFn: fetchLeads })
      .catch(console.error), // v5 lança — não deixar crashar o render
    queryClient.prefetchQuery({ queryKey: ['stats'], queryFn: fetchStats })
      .catch(console.error),
  ])

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <LeadsTable />
      <StatsWidget />
    </HydrationBoundary>
  )
}

// Client Component — recebe dados do cache hidratado
'use client'
export function LeadsTable() {
  const { data, isPending, isError } = useQuery({
    queryKey: ['leads'],
    queryFn: fetchLeads,
    staleTime: 60 * 1000, // OBRIGATÓRIO: >= staleTime do servidor
  })
  if (isPending) return <TableSkeleton />
  if (isError) return <ErrorState />
  return <DataTable columns={columns} data={data} />
}
```

### Mutações com invalidação

```tsx
'use client'
import { useMutation, useQueryClient } from '@tanstack/react-query'

export function useCreateLead() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (data: NewLead) => createLeadAction(data),
    onSuccess: () => {
      // Invalida cache → TanStack refaz o fetch automaticamente
      queryClient.invalidateQueries({ queryKey: ['leads'] })
    },
    onError: (error) => {
      toast.error('Erro ao criar lead')
    },
  })
}
```

### Optimistic Updates

```tsx
return useMutation({
  mutationFn: updateLeadStatus,
  onMutate: async (newStatus) => {
    await queryClient.cancelQueries({ queryKey: ['leads'] })
    const previous = queryClient.getQueryData(['leads'])
    queryClient.setQueryData(['leads'], (old: Lead[]) =>
      old.map((l) => l.id === newStatus.id ? { ...l, status: newStatus.status } : l)
    )
    return { previous }
  },
  onError: (err, _, context) => {
    queryClient.setQueryData(['leads'], context?.previous)
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['leads'] })
  },
})
```

### v4 → v5 Breaking Changes Resumidas

```tsx
// v4 — REMOVIDO em v5
useQuery(['posts'], fetchPosts, { onSuccess: (data) => ... })

// v5 — correto
const { data } = useQuery({ queryKey: ['posts'], queryFn: fetchPosts })
useEffect(() => { if (data) console.log(data) }, [data])

// v4
<Hydrate state={...}>
// v5
<HydrationBoundary state={...}>

// isPending substitui isLoading para "sem dados ainda"
const { data, isPending } = useQuery(...)
```

---

## Zustand v5 — Factory Pattern Completo

### Store factory

```ts
// lib/store/ui-store.ts
import { createStore } from 'zustand/vanilla'

export type UiState = {
  sidebarOpen: boolean
  activeModule: string | null
  selectedLeadId: string | null
}

export type UiActions = {
  toggleSidebar: () => void
  setActiveModule: (module: string) => void
  selectLead: (id: string | null) => void
  reset: () => void
}

export type UiStore = UiState & UiActions

const defaultState: UiState = {
  sidebarOpen: true,
  activeModule: null,
  selectedLeadId: null,
}

export function createUiStore(initState: Partial<UiState> = {}) {
  return createStore<UiStore>()((set) => ({
    ...defaultState,
    ...initState,
    toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
    setActiveModule: (module) => set({ activeModule: module }),
    selectLead: (id) => set({ selectedLeadId: id }),
    reset: () => set(defaultState),
  }))
}

export type UiStoreApi = ReturnType<typeof createUiStore>
```

### Provider

```tsx
// providers/ui-store-provider.tsx
'use client'
import { createContext, useContext, useRef, type ReactNode } from 'react'
import { useStore } from 'zustand'
import { createUiStore, type UiStore, type UiStoreApi } from '@/lib/store/ui-store'

const UiStoreContext = createContext<UiStoreApi | null>(null)

export function UiStoreProvider({
  children,
  initialState,
}: {
  children: ReactNode
  initialState?: Partial<UiStore>
}) {
  const storeRef = useRef<UiStoreApi | null>(null)
  if (!storeRef.current) {
    storeRef.current = createUiStore(initialState)
  }
  return <UiStoreContext.Provider value={storeRef.current}>{children}</UiStoreContext.Provider>
}

export function useUiStore<T>(selector: (store: UiStore) => T): T {
  const ctx = useContext(UiStoreContext)
  if (!ctx) throw new Error('useUiStore must be used within UiStoreProvider')
  return useStore(ctx, selector)
}
```

### Consumo eficiente — selector pattern

```tsx
'use client'
import { useShallow } from 'zustand/react/shallow' // v5: novo path

// Selector simples — re-render só quando sidebarOpen muda
const sidebarOpen = useUiStore((s) => s.sidebarOpen)

// Múltiplos valores — useShallow evita over-rendering
const { sidebarOpen, activeModule } = useUiStore(
  useShallow((s) => ({ sidebarOpen: s.sidebarOpen, activeModule: s.activeModule }))
)

// Apenas action — estável, nunca muda, sem re-render
const toggleSidebar = useUiStore((s) => s.toggleSidebar)
```

### Zustand com persist (preferências do usuário)

```ts
import { createStore } from 'zustand/vanilla'
import { persist, createJSONStorage } from 'zustand/middleware'

export function createPrefsStore() {
  return createStore<PrefsStore>()(
    persist(
      (set) => ({ theme: 'light', setTheme: (t) => set({ theme: t }) }),
      { name: 'triforce-prefs', storage: createJSONStorage(() => localStorage) }
    )
  )
}
```

---

## Fontes
- https://tanstack.com/query/v5/docs/framework/react/guides/advanced-ssr
- https://tanstack.com/query/v5/docs/framework/react/guides/migrating-to-v5
- https://zustand.docs.pmnd.rs/guides/nextjs
- https://zustand.docs.pmnd.rs/guides/initialize-state-with-props
