# Performance + DataTable + Acessibilidade

## Core Web Vitals — Next.js 15 App Router

### LCP (Largest Contentful Paint) — meta: < 2.5s

```tsx
// Uma única imagem above-the-fold com priority
// sizes é obrigatório com fill — omitir causa download 100vw desnecessário
<Image
  src={heroImage}
  alt="Descrição"
  fill
  priority                  // preload no <head>
  fetchPriority="high"      // hint pro browser
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 40vw"
  className="object-cover"
/>

// Wrapper obrigatório com position: relative e dimensões
<div className="relative h-64 w-full">
  <Image src={img} alt="..." fill sizes="100vw" priority />
</div>
```

**Causas comuns de LCP ruim:**
- Imagem sem `priority` → carregada depois do JS
- `fill` sem `sizes` → browser baixa versão 100vw sempre
- Fonte bloqueando render (usar `display: 'swap'`)

### CLS (Cumulative Layout Shift) — meta: < 0.1

```tsx
// SEMPRE width + height explícitos em imagens não-fill
<Image src={avatar} alt="..." width={48} height={48} />

// Ou wrapper com aspect-ratio (evita reflow quando imagem carrega)
<div className="relative aspect-video w-full">
  <Image src={thumbnail} alt="..." fill sizes="..." />
</div>

// Fontes: adjustFontFallback elimina shift do font swap
import { Inter } from 'next/font/google'
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  adjustFontFallback: true, // métrica-matched fallback → CLS = 0
})

// Variável CSS para Tailwind
const geist = Geist({
  subsets: ['latin'],
  variable: '--font-geist', // usar como className={geist.variable}
})
```

### INP (Interaction to Next Paint) — meta: < 200ms

```tsx
// Filtros/buscas pesadas: useTransition para não bloquear UI
const [isPending, startTransition] = useTransition()

function handleFilterChange(value: string) {
  startTransition(() => {
    setFilter(value) // não bloqueia input/scroll durante update
  })
}

// Lista grande renderizada: useDeferredValue
const deferredQuery = useDeferredValue(searchQuery)
const filteredLeads = useMemo(
  () => leads.filter((l) => l.nome.includes(deferredQuery)),
  [leads, deferredQuery]
)

// Operações pesadas fora do render thread
function handleHeavyOperation() {
  startTransition(async () => {
    await processData() // browser pode respirar durante processamento
    setResult(data)
  })
}
```

### Bundle e Dynamic Imports

```tsx
// Componentes pesados: lazy load com fallback
import dynamic from 'next/dynamic'

const HeavyChart = dynamic(() => import('@/components/charts/heavy-chart'), {
  loading: () => <Skeleton className="h-64 w-full" />,
  ssr: false, // se usa Browser APIs (window, document)
})

const RichTextEditor = dynamic(() => import('@/components/editor'), {
  loading: () => <Skeleton className="h-40 w-full" />,
  ssr: false,
})

// Named exports precisam de path explícito
const { Chart } = dynamic(
  () => import('@/components/charts').then((mod) => ({ default: mod.Chart }))
)
```

```tsx
// next/script: estratégias de carregamento
import Script from 'next/script'

// afterInteractive: após hydration (analytics, chat)
<Script src="https://analytics.example.com/script.js" strategy="afterInteractive" />

// lazyOnload: idle (low priority)
<Script src="..." strategy="lazyOnload" />

// beforeInteractive: critical (raramente necessário)
<Script src="..." strategy="beforeInteractive" />
```

### next/font — Setup correto

```tsx
// Fonte Google com variável CSS + Tailwind
import { Inter, Roboto_Mono } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
  adjustFontFallback: true,
})

// Fonte local com múltiplos pesos
import localFont from 'next/font/local'
const customFont = localFont({
  src: [
    { path: '../fonts/custom-400.woff2', weight: '400' },
    { path: '../fonts/custom-700.woff2', weight: '700' },
  ],
  variable: '--font-custom',
  display: 'swap',
})

// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="pt-BR" className={`${inter.variable} ${customFont.variable}`}>
      <body className="font-inter">{children}</body>
    </html>
  )
}
```

---

## DataTable — TanStack Table v8 + shadcn/ui

### ColumnDef tipada

```tsx
// components/leads/columns.tsx
import { type ColumnDef } from '@tanstack/react-table'
import { type Lead } from '@/db/schema/crm'

export const leadColumns: ColumnDef<Lead>[] = [
  {
    id: 'select',
    header: ({ table }) => (
      <Checkbox
        checked={table.getIsAllPageRowsSelected()}
        onCheckedChange={(v) => table.toggleAllPageRowsSelected(!!v)}
        aria-label="Selecionar todos"
      />
    ),
    cell: ({ row }) => (
      <Checkbox
        checked={row.getIsSelected()}
        onCheckedChange={(v) => row.toggleSelected(!!v)}
        aria-label="Selecionar linha"
      />
    ),
    enableSorting: false,
  },
  {
    accessorKey: 'nome',
    header: ({ column }) => (
      <Button variant="ghost" onClick={() => column.toggleSorting(column.getIsSorted() === 'asc')}>
        Nome <ArrowUpDown className="ml-2 h-4 w-4" />
      </Button>
    ),
  },
  {
    accessorKey: 'status',
    header: 'Status',
    cell: ({ row }) => <Badge>{row.getValue('status')}</Badge>,
  },
  {
    id: 'actions',
    cell: ({ row }) => <LeadRowActions lead={row.original} />,
  },
]
```

### Client-side DataTable (< 500 registros)

```tsx
// components/data-table.tsx
'use client'
import {
  useReactTable, getCoreRowModel, getSortedRowModel,
  getFilteredRowModel, getPaginationRowModel, flexRender,
  type ColumnDef, type SortingState, type VisibilityState,
} from '@tanstack/react-table'

type DataTableProps<TData> = {
  columns: ColumnDef<TData>[]
  data: TData[]
}

export function DataTable<TData>({ columns, data }: DataTableProps<TData>) {
  const [sorting, setSorting] = useState<SortingState>([])
  const [globalFilter, setGlobalFilter] = useState('')
  const [columnVisibility, setColumnVisibility] = useState<VisibilityState>({})
  const [rowSelection, setRowSelection] = useState({})

  const table = useReactTable({
    data,
    columns,
    state: { sorting, globalFilter, columnVisibility, rowSelection },
    onSortingChange: setSorting,
    onGlobalFilterChange: setGlobalFilter,
    onColumnVisibilityChange: setColumnVisibility,
    onRowSelectionChange: setRowSelection,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    enableRowSelection: true,
  })

  const selectedRows = table.getFilteredSelectedRowModel().rows.map((r) => r.original)

  return (
    <div className="space-y-4">
      {/* Toolbar */}
      <div className="flex items-center gap-2">
        <Input
          placeholder="Buscar..."
          value={globalFilter}
          onChange={(e) => setGlobalFilter(e.target.value)}
          className="max-w-sm"
        />
        {/* Visibilidade de colunas */}
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="outline">Colunas</Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent>
            {table.getAllColumns().filter((c) => c.getCanHide()).map((column) => (
              <DropdownMenuCheckboxItem
                key={column.id}
                checked={column.getIsVisible()}
                onCheckedChange={(v) => column.toggleVisibility(!!v)}
              >
                {column.id}
              </DropdownMenuCheckboxItem>
            ))}
          </DropdownMenuContent>
        </DropdownMenu>

        {/* Bulk actions */}
        {selectedRows.length > 0 && (
          <Button variant="destructive" size="sm" onClick={() => handleBulkDelete(selectedRows)}>
            Excluir {selectedRows.length} selecionados
          </Button>
        )}
      </div>

      {/* Table */}
      <Table role="grid">
        <TableHeader>
          {table.getHeaderGroups().map((hg) => (
            <TableRow key={hg.id}>
              {hg.headers.map((h) => (
                <TableHead key={h.id} role="columnheader"
                  aria-sort={h.column.getIsSorted() === 'asc' ? 'ascending'
                    : h.column.getIsSorted() === 'desc' ? 'descending' : 'none'}>
                  {h.isPlaceholder ? null : flexRender(h.column.columnDef.header, h.getContext())}
                </TableHead>
              ))}
            </TableRow>
          ))}
        </TableHeader>
        <TableBody>
          {table.getRowModel().rows.length ? (
            table.getRowModel().rows.map((row) => (
              <TableRow key={row.id} data-state={row.getIsSelected() && 'selected'}
                role="row" aria-selected={row.getIsSelected()}>
                {row.getVisibleCells().map((cell) => (
                  <TableCell key={cell.id} role="gridcell">
                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                  </TableCell>
                ))}
              </TableRow>
            ))
          ) : (
            <TableRow><TableCell colSpan={columns.length} className="text-center">Nenhum resultado.</TableCell></TableRow>
          )}
        </TableBody>
      </Table>

      {/* Pagination */}
      <div className="flex items-center justify-between">
        <p className="text-sm text-muted-foreground">
          {table.getFilteredSelectedRowModel().rows.length} de{' '}
          {table.getFilteredRowModel().rows.length} linha(s) selecionada(s)
        </p>
        <div className="flex items-center gap-2">
          <Button variant="outline" size="sm" onClick={() => table.previousPage()} disabled={!table.getCanPreviousPage()}>Anterior</Button>
          <span className="text-sm">Página {table.getState().pagination.pageIndex + 1} de {table.getPageCount()}</span>
          <Button variant="outline" size="sm" onClick={() => table.nextPage()} disabled={!table.getCanNextPage()}>Próxima</Button>
        </div>
      </div>
    </div>
  )
}
```

### Server-side DataTable (> 500 registros ou filtros complexos)

```tsx
// Server-side: manualPagination + URL sync + TanStack Query
'use client'
import { useSearchParams, useRouter, usePathname } from 'next/navigation'
import { useQuery } from '@tanstack/react-query'
import { keepPreviousData } from '@tanstack/react-query'

export function ServerDataTable<TData>({ columns }: { columns: ColumnDef<TData>[] }) {
  const searchParams = useSearchParams()
  const router = useRouter()
  const pathname = usePathname()

  const page = Number(searchParams.get('page') ?? '0')
  const sort = searchParams.get('sort') ?? 'nome'
  const order = (searchParams.get('order') ?? 'asc') as 'asc' | 'desc'
  const filter = searchParams.get('q') ?? ''

  // queryKey inclui todo o estado → refetch automático ao mudar filtros
  const { data, isPending } = useQuery({
    queryKey: ['leads', { page, sort, order, filter }],
    queryFn: () => fetchLeadsPaginated({ page, sort, order, filter }),
    placeholderData: keepPreviousData, // sem flash ao paginar
  })

  const table = useReactTable({
    data: data?.rows ?? [],
    columns,
    rowCount: data?.total ?? 0,
    state: {
      pagination: { pageIndex: page, pageSize: 20 },
      sorting: [{ id: sort, desc: order === 'desc' }],
    },
    manualPagination: true,
    manualSorting: true,
    manualFiltering: true,
    onPaginationChange: (updater) => {
      const next = typeof updater === 'function'
        ? updater({ pageIndex: page, pageSize: 20 })
        : updater
      const params = new URLSearchParams(searchParams)
      params.set('page', String(next.pageIndex))
      startTransition(() => router.push(`${pathname}?${params}`))
    },
    // ...
    getCoreRowModel: getCoreRowModel(),
  })

  return <>{/* renderizar tabela */}</>
}
```

---

## Acessibilidade (a11y) — WCAG 2.1 AA

### Skip navigation

```tsx
// app/layout.tsx — primeiro elemento focável da página
<a
  href="#main-content"
  className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 focus:z-50
             focus:rounded focus:bg-background focus:px-4 focus:py-2 focus:shadow-md"
>
  Pular para o conteúdo principal
</a>
<main id="main-content" tabIndex={-1}> {/* tabIndex=-1 para receber foco via href */}
  {children}
</main>
```

### Live regions para dados assíncronos

```tsx
// Atualizar region quando dados chegam — sr vocaliza automaticamente
// IMPORTANTE: manter o elemento no DOM sempre (não conditional render)
<div role="status" aria-live="polite" aria-atomic="true" className="sr-only">
  {isPending ? 'Carregando leads...' : `${data?.length ?? 0} leads encontrados`}
</div>

// aria-live="assertive" apenas para erros críticos (interrompe qualquer leitura)
<div role="alert" aria-live="assertive" className="sr-only">
  {error ? `Erro: ${error.message}` : ''}
</div>
```

### Focus management após navegação

```tsx
// components/focus-manager.tsx — restaurar foco após mudança de rota
'use client'
import { usePathname } from 'next/navigation'
import { useEffect, useRef } from 'react'

export function FocusManager() {
  const pathname = usePathname()
  const announcerRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    // Mover foco para o topo após navegação
    announcerRef.current?.focus()
  }, [pathname])

  return (
    <div ref={announcerRef} tabIndex={-1} className="sr-only" aria-live="polite">
      {/* Conteúdo da página anunciado pelo RouteAnnouncer do Next.js */}
    </div>
  )
}
```

### Focus trap em modais

```tsx
// Usar Dialog do shadcn/ui — já implementa focus trap via Radix UI
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog'

// NÃO reimplementar focus trap manualmente — Radix já trata:
// - Foco vai para o Dialog ao abrir
// - Tab fica preso dentro do Dialog
// - Fecha com Escape
// - Foco volta ao trigger ao fechar
<Dialog open={open} onOpenChange={setOpen}>
  <DialogContent>
    <DialogHeader><DialogTitle>Criar Lead</DialogTitle></DialogHeader>
    <CreateLeadForm onSuccess={() => setOpen(false)} />
  </DialogContent>
</Dialog>
```

### Contraste WCAG AA

| Tipo de elemento | Contraste mínimo |
|-----------------|-----------------|
| Texto normal (< 18pt) | 4.5:1 |
| Texto grande (≥ 18pt ou 14pt bold) | 3:1 |
| Elementos UI (bordas, ícones) | 3:1 |
| Texto desabilitado | Isento |

shadcn/ui dark/light themes já atingem AA por padrão — não alterar sem verificar contraste.

Ferramentas: `@axe-core/react` em dev, Chrome DevTools Accessibility panel.

```tsx
// Integrar axe apenas em dev
if (process.env.NODE_ENV === 'development') {
  import('@axe-core/react').then(({ default: axe }) => {
    axe(React, ReactDOM, 1000)
  })
}
```

### `prefers-reduced-motion`

```css
/* globals.css — reduzir todas as animações para quem prefere */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

```tsx
// Tailwind: motion-safe: e motion-reduce: variants
<div className="transition-all motion-reduce:transition-none duration-200">
  ...
</div>
```

---

## Fontes
- https://nextjs.org/docs/app/api-reference/components/image
- https://nextjs.org/docs/app/api-reference/components/font
- https://tanstack.com/table/v8/docs/guide/column-defs
- https://tanstack.com/table/v8/docs/guide/pagination#manual-server-side-pagination
- https://www.w3.org/WAI/WCAG21/quickref/
- https://www.w3.org/WAI/ARIA/apg/patterns/grid/
- https://web.dev/articles/inp
