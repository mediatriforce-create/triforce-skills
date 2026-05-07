# Arquitetura do Sistema Interno — Triforce Auto

## Visão Geral

Sistema web interno da Triforce Auto com 4 módulos integrados e conexão com o CRM Airtable existente.

**Stack:** Next.js 15 (App Router) + TypeScript + Supabase (Postgres + Auth + RLS) + Drizzle ORM + Vercel + Airtable API

**Modelo:** Single-tenant, single Supabase project, single Next.js deployment.

---

## Módulos

| Módulo | Path | Tabelas Supabase | Airtable sync |
|--------|------|-----------------|---------------|
| CRM | `/crm` | `leads`, `clientes`, `interacoes` | Sim — bidirectional via webhook |
| Financeiro | `/financeiro` | `extratos`, `faturas`, `transacoes` | Não |
| Gestão de Time | `/time` | `funcionarios`, `cargos`, `equipes` | Não |
| Tarefas | `/tarefas` | `tarefas`, `projetos`, `comentarios` | Não |

---

## Estrutura de Pastas

```
src/
  app/
    (auth)/
      login/
        page.tsx
      recuperar-senha/
        page.tsx
    (app)/
      layout.tsx          ← shell global: sidebar, nav, verificação de sessão
      crm/
        layout.tsx         ← nav CRM
        page.tsx           ← /crm dashboard
        leads/
          page.tsx
          [id]/
            page.tsx
          _components/     ← componentes privados de leads
          _actions/        ← Server Actions de leads
        clientes/
        interacoes/
      financeiro/
        layout.tsx
        extrato/
          page.tsx
        faturas/
      time/
        layout.tsx
        funcionarios/
      tarefas/
        layout.tsx
        page.tsx
        [projetoId]/
    api/
      airtable/
        leads/
          route.ts         ← GET cached (runtime: edge)
      webhooks/
        airtable/
          route.ts         ← POST: Airtable → Supabase sync
    globals.css
    layout.tsx             ← root layout
  components/              ← UI compartilhada (DataTable, Modal, Badge, etc.)
    ui/                    ← shadcn/ui components
    layout/                ← Sidebar, Header, Breadcrumb
  db/
    client.ts              ← Drizzle client (pooler + direct)
    rls.ts                 ← RLS client (wraps em transaction)
    schema/
      index.ts
      crm.ts
      financeiro.ts
      time.ts
      tarefas.ts
      shared.ts
  lib/
    airtable/
      client.ts
      cache.ts
      queue.ts
    supabase/
      server.ts
      client.ts
    auth/
      permissions.ts
  middleware.ts
docs/
  adr/                     ← Architecture Decision Records
    ADR-001-stack-nextjs-supabase.md
    ADR-002-drizzle-vs-supabase-client.md
    ADR-003-airtable-sync-strategy.md
```

---

## Estado — Regras de Ouro

**Server data (fetch, Supabase, Airtable):** TanStack Query
- Prefetch no servidor em Server Components
- Hidratação no cliente
- Invalidação por tag após mutações

**UI state global (sidebar, filtros, seleção):** Zustand
- Store factory com `createStore` (evitar SSR singleton leak)
- Provider no `(app)/layout.tsx`

**Form state:** React Hook Form + Zod
- Schema derivado de `drizzle-zod` como single source of truth

**Nunca usar:** Redux, Context para server data, useState para dados do servidor.

---

## Auth + RBAC

```ts
// Roles definidos
type Role = 'admin' | 'gestor' | 'operador' | 'visualizador'

// Permissões por módulo
const permissions = {
  crm: {
    admin: ['read', 'write', 'delete'],
    gestor: ['read', 'write'],
    operador: ['read', 'write'],
    visualizador: ['read'],
  },
  financeiro: {
    admin: ['read', 'write', 'delete'],
    gestor: ['read'],
    operador: [],
    visualizador: [],
  },
  // ...
}
```

**Camadas de proteção:**
1. `middleware.ts` — bloqueia `/(app)/*` sem sessão válida
2. `(app)/layout.tsx` — re-verifica sessão server-side, redireciona se expirada
3. Server Actions — verifica role antes de qualquer mutação
4. RLS Supabase — enforcement no banco (última linha de defesa)

---

## Padrão de Componentes

**Server Components por padrão.** Usar `'use client'` apenas quando necessário:
- Interatividade (onClick, onChange, formulários)
- Browser APIs (localStorage, window)
- TanStack Query hooks

**Data fetching:** Server Components fazem o fetch inicial. Componentes client hidratam com TanStack Query.

```tsx
// app/(app)/crm/leads/page.tsx — Server Component
import { getCachedLeads } from '@/lib/airtable/cache'

export default async function LeadsPage() {
  const leads = await getCachedLeads() // fetch no servidor, cacheado
  return <LeadsTable initialData={leads} />
}

// components/LeadsTable.tsx — Client Component (TanStack Query)
'use client'
import { useQuery } from '@tanstack/react-query'

export function LeadsTable({ initialData }) {
  const { data } = useQuery({
    queryKey: ['leads'],
    queryFn: fetchLeads,
    initialData, // hidrata com dados do servidor
  })
  // ...
}
```

---

## ADRs Obrigatórios (criar no início do projeto)

1. **ADR-001** — Escolha do stack (Next.js 15 + Supabase + Drizzle)
2. **ADR-002** — Drizzle ORM vs Supabase client direto para queries
3. **ADR-003** — Estratégia de sync Airtable (FDW + webhooks)
4. **ADR-004** — Estado: TanStack Query + Zustand
5. **ADR-005** — RBAC: Supabase Auth custom claims + RLS

---

## Checklist de Setup Inicial

- [ ] Supabase project criado na região **São Paulo (sa-east-1)**
- [ ] `DATABASE_URL` (pooler Transaction porta 6543) configurado
- [ ] `DIRECT_URL` (porta 5432 direta) configurado para migrations
- [ ] `AIRTABLE_API_KEY` e `AIRTABLE_BASE_ID` (appDxa9P3sBnuNNc3) configurados
- [ ] Drizzle Kit configurado com `DIRECT_URL`
- [ ] Middleware de auth configurado em `middleware.ts`
- [ ] Supabase Auth: Site URL e redirect URLs configurados
- [ ] GitHub repo criado + branch protection em `main`
- [ ] Vercel project criado + env vars configuradas
- [ ] CI/CD GitHub Actions: lint + typecheck + build + migrate
- [ ] ADR-001 criado em `docs/adr/`
