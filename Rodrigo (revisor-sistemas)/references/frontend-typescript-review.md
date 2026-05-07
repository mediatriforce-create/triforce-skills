# Frontend + TypeScript — Guia de Revisão

## Next.js 15 App Router — Checklist de Revisão

### `'use client'` — justificativa obrigatória

```tsx
// 🟡 OBRIGATÓRIO — 'use client' sem necessidade degrada performance
'use client'
export function UserCard({ user }: { user: User }) {
  return (
    <div className="rounded-md border p-4">
      <h3>{user.nome}</h3>
      <p>{user.email}</p>
    </div>
  )
  // Sem hooks, sem event handlers, sem Browser APIs → Server Component
}

// ✅ CORRETO — Server Component
export function UserCard({ user }: { user: User }) {
  return (
    <div className="rounded-md border p-4">
      <h3>{user.nome}</h3>
      <p>{user.email}</p>
    </div>
  )
}

// ✅ CORRETO — extrair só a parte interativa
export function UserCard({ user }: { user: User }) {
  return (
    <div className="rounded-md border p-4">
      <h3>{user.nome}</h3>
      <UserActions userId={user.id} /> {/* este sim é client */}
    </div>
  )
}
```

**Perguntas obrigatórias para todo `'use client'`:**
1. O componente usa `useState`, `useEffect`, `useRef` ou outro hook React?
2. Tem event handlers (`onClick`, `onChange`, `onSubmit`)?
3. Acessa Browser APIs (`window`, `localStorage`, `document`)?
4. Usa hooks de bibliotecas client-only (TanStack Query hooks, Zustand)?

Se a resposta for **não para todas**: `'use client'` não é necessário.

### Hydration Safety — 4 padrões de risco

```tsx
// 🔴 BLOQUEANTE — Date.now() difere entre server e client → crash de hydration
export function LastSeen() {
  return <span>Visto há {Math.floor((Date.now() - user.lastSeen) / 1000)}s</span>
}

// 🔴 BLOQUEANTE — Math.random() não-determinístico
export function Avatar() {
  return <div style={{ background: `hsl(${Math.random() * 360}, 70%, 50%)` }} />
}

// 🔴 BLOQUEANTE — window não existe no servidor
export function ScrollPosition() {
  return <span>Posição: {window.scrollY}px</span>
}

// 🔴 BLOQUEANTE — localStorage não existe no servidor
export function Theme() {
  return <div className={localStorage.getItem('theme') ?? 'light'} />
}

// ✅ CORRETO — todos via useEffect (roda só no client)
'use client'
export function LastSeen({ lastSeenAt }: { lastSeenAt: number }) {
  const [ago, setAgo] = useState<string | null>(null)
  useEffect(() => {
    setAgo(`${Math.floor((Date.now() - lastSeenAt) / 1000)}s atrás`)
  }, [lastSeenAt])
  return <span>{ago ?? '...'}</span>
}
```

### Async APIs do Next.js 15

```tsx
// 🟡 OBRIGATÓRIO — params/searchParams sem await causam runtime error
export default function LeadPage({ params }: { params: { id: string } }) {
  const { id } = params // ERRO em Next.js 15 — params é uma Promise
}

// ✅ CORRETO
export default async function LeadPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
}

// Mesmo para cookies() e headers()
import { cookies } from 'next/headers'
const cookieStore = await cookies() // await obrigatório
```

---

## TanStack Query v5 — Padrões de Review

### Cache invalidation após mutations

```tsx
// 🟡 OBRIGATÓRIO — sem invalidação, UI mostra dado stale
'use client'
const { mutate: updateStatus } = useMutation({
  mutationFn: (data: UpdateStatusInput) => updateLeadStatusAction(data),
  // sem onSuccess → tabela mostra status antigo até refresh manual
})

// ✅ CORRETO
const queryClient = useQueryClient()
const { mutate: updateStatus } = useMutation({
  mutationFn: (data: UpdateStatusInput) => updateLeadStatusAction(data),
  onSuccess: () => {
    // Invalida todas as queries de leads (cache é refetchado automaticamente)
    queryClient.invalidateQueries({ queryKey: ['leads'] })
    // Se a mutation afeta uma entidade específica, invalide também:
    queryClient.invalidateQueries({ queryKey: ['lead', data.id] })
  },
  onError: (error) => {
    toast.error('Erro ao atualizar status')
  },
})
```

### staleTime deve ser > 0 para prefetch SSR

```tsx
// 🟡 OBRIGATÓRIO — staleTime = 0 (padrão) refetch imediato no mount
// O prefetch do servidor é descartado na hydration
const { data } = useQuery({
  queryKey: ['leads'],
  queryFn: fetchLeads,
  // sem staleTime → refetch imediato → prefetch desperdiçado
})

// ✅ CORRETO — deve ser >= staleTime do servidor (getQueryClient())
const { data } = useQuery({
  queryKey: ['leads'],
  queryFn: fetchLeads,
  staleTime: 60 * 1000, // 1 minuto — corresponde ao servidor
})
```

### HydrationBoundary (v5) — não Hydrate (v4)

```tsx
// 🔴 BLOQUEANTE — componente do v4, não existe no v5
import { Hydrate } from '@tanstack/react-query' // REMOVIDO em v5

// ✅ CORRETO — v5
import { HydrationBoundary, dehydrate } from '@tanstack/react-query'

export default async function Page() {
  const queryClient = getQueryClient()
  await queryClient.prefetchQuery({ queryKey: ['leads'], queryFn: fetchLeads })
    .catch(console.error) // v5 lança em erro — não deixar crashar o render
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <LeadsTable />
    </HydrationBoundary>
  )
}
```

---

## Zustand — Anti-patterns de Review

### Dados do servidor no Zustand

```tsx
// 🔴 BLOQUEANTE — reinventa TanStack Query de forma pior
// Sem cache, sem deduplication, sem background refresh, sem SSR hydration
'use client'
const setLeads = useLeadsStore((s) => s.setLeads)
const leads = useLeadsStore((s) => s.leads)

useEffect(() => {
  fetchLeads().then(setLeads) // toda vez que o componente monta
}, [])

// ✅ CORRETO — server data sempre via TanStack Query
const { data: leads, isPending } = useQuery({
  queryKey: ['leads'],
  queryFn: fetchLeads,
  staleTime: 60_000,
})
```

### Singleton de módulo (SSR leak)

```ts
// 🔴 BLOQUEANTE — singleton no App Router vaza estado entre requests
import { create } from 'zustand'
export const useLeadsStore = create<LeadsState>()((set) => ({ ... }))
// Usuário A contamina estado de Usuário B no servidor

// ✅ CORRETO — factory + provider (ver arquitetura-sistema.md)
export function createLeadsStore() {
  return createStore<LeadsState>()((set) => ({ ... }))
}
```

---

## TypeScript — Checklist de Review

### `any` — quando flaggar vs aceitar

```ts
// 🟡 OBRIGATÓRIO — any em superfície pública (parâmetro de função exportada)
export async function processLead(data: any) { ... }
// Quem chama não tem nenhuma garantia de tipo

// ✅ — Zod + TypeScript
export async function processLead(data: z.infer<typeof leadSchema>) { ... }

// 🟢 SUGESTÃO (não bloqueante) — any em adaptador interno fino
// Ex: tipagem de biblioteca externa mal tipada
const result = (thirdPartyLib as any).weirdMethod()
// Aceitável se está isolado e tem comentário justificando
```

### Type assertions em dados externos

```ts
// 🔴 BLOQUEANTE — type assertion sem validação em dado externo
const payload = req.body as Lead // payload de webhook, API, form — nunca tem tipo garantido
// Se o shape mudou no provider externo → crash em runtime

// ✅ CORRETO — Zod valida e infere
const result = leadSchema.safeParse(await req.json())
if (!result.success) {
  return Response.json({ error: result.error.flatten() }, { status: 422 })
}
const payload = result.data // tipo Lead garantido pelo Zod
```

### Retornos de Server Actions

```ts
// 🟡 OBRIGATÓRIO — type implícito em Server Action abre brecha para `any`
export const createLeadAction = authedActionClient
  .schema(createLeadSchema)
  .action(async ({ parsedInput, ctx }) => {
    const lead = await db.insert(leads).values(parsedInput).returning()
    return lead[0] // tipo correto por inferência, mas sem garantia em erros
  })

// ✅ CORRETO — discriminated union para retorno explícito
type ActionResult<T> =
  | { success: true; data: T }
  | { success: false; code: string; message: string }

.action(async ({ parsedInput, ctx }): Promise<ActionResult<Lead>> => {
  try {
    const [lead] = await db.insert(leads).values(parsedInput).returning()
    return { success: true, data: lead }
  } catch (e) {
    return { success: false, code: 'DB_ERROR', message: 'Erro ao criar lead' }
  }
})
```

### Zod schema como single source of truth

```ts
// 🟡 OBRIGATÓRIO — schema duplicado diverge silenciosamente
// Schema no action:
const createLeadSchema = z.object({ nome: z.string().min(1), telefone: z.string() })
// Schema no form (diferente!):
const formSchema = z.object({ nome: z.string(), phone: z.string() }) // campo renomeado!

// ✅ CORRETO — um schema, importado em ambos
// lib/schemas/leads.ts
export const createLeadSchema = createInsertSchema(leads, { ... })
export type CreateLeadValues = z.infer<typeof createLeadSchema>

// No form: import { createLeadSchema, CreateLeadValues }
// No action: import { createLeadSchema }
// Mesma fonte → nunca divergem
```

### Checklist TypeScript por PR

- [ ] Sem `any` em parâmetros ou retornos de funções exportadas?
- [ ] Type assertions (`as Type`) apenas com Zod validate ou `instanceof` guard?
- [ ] Server Actions com tipo de retorno explícito?
- [ ] Schemas Zod derivados de `drizzle-zod` quando possível (single source of truth)?
- [ ] `z.infer<typeof schema>` para tipos derivados (não interface duplicada)?
- [ ] `safeParse` (não `parse`) em dados externos — sem throw não-tratado?

---

## Fontes
- https://nextjs.org/docs/app/guides/upgrading/version-15
- https://tanstack.com/query/v5/docs/framework/react/guides/mutations
- https://zustand.docs.pmnd.rs/guides/nextjs
- https://zod.dev/error-formatting
- https://www.typescriptlang.org/docs/handbook/2/narrowing.html
