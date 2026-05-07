# Drizzle ORM + Supabase — Referência Completa

> Fonte: Drizzle docs oficiais + Supabase docs + rphlmr/drizzle-supabase-rls

## Conexão — Padrão Obrigatório

```ts
// db/client.ts — Next.js 15 / Vercel serverless
import { drizzle } from 'drizzle-orm/postgres-js'
import postgres from 'postgres'
import * as schema from './schema'

// APP: Transaction Pooler (porta 6543)
const pool = postgres(process.env.DATABASE_URL!, {
  prepare: false, // OBRIGATÓRIO — Transaction mode não suporta prepared statements
  max: 1,         // serverless: uma conexão por invocação
})
export const db = drizzle(pool, { schema })
```

```ts
// drizzle.config.ts — Migrations sempre via Direct URL
import { defineConfig } from 'drizzle-kit'

export default defineConfig({
  schema: './src/db/schema',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DIRECT_URL!, // porta 5432 direta — NUNCA pooler
  },
})
```

**Variáveis de ambiente:**
```
DATABASE_URL=postgresql://postgres.[ref]:[pass]@aws-0-sa-east-1.pooler.supabase.com:6543/postgres
DIRECT_URL=postgresql://postgres:[pass]@db.[ref].supabase.co:5432/postgres
```

---

## RLS com Drizzle — Padrão rphlmr

RLS requer injetar o JWT do usuário na sessão Postgres. Sem isso, Drizzle vê `anon` e RLS bloqueia tudo.

```ts
// db/rls.ts
import { drizzle } from 'drizzle-orm/postgres-js'
import postgres from 'postgres'
import { sql } from 'drizzle-orm'
import * as schema from './schema'

// Admin: bypassa RLS (service_role)
const adminPool = postgres(process.env.DIRECT_URL!, { prepare: false })
export const adminDb = drizzle(adminPool, { schema })

// RLS: usa usuário restrito com grants anon/authenticated
const rlsPool = postgres(process.env.DATABASE_URL!, { prepare: false })
const baseDb = drizzle(rlsPool, { schema })

export async function createRlsClient(accessToken: string) {
  return baseDb.transaction(async (tx) => {
    // set_config com is_local=true persiste APENAS dentro desta transaction
    await tx.execute(sql`
      SELECT
        set_config('request.jwt', ${accessToken}, true),
        set_config('request.jwt.claim.sub', ${parseJwtSub(accessToken)}, true),
        set_role('authenticated')
    `)
    return tx
  })
}

function parseJwtSub(token: string): string {
  const payload = JSON.parse(atob(token.split('.')[1]))
  return payload.sub
}
```

**Uso em Server Actions:**
```ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'
import { createRlsClient } from '@/db/rls'

export async function getLeads() {
  const cookieStore = await cookies()
  const supabase = createServerClient(...)
  const { data: { session } } = await supabase.auth.getSession()

  const rls = await createRlsClient(session!.access_token)
  // Todas as queries aqui respeitam RLS do usuário autenticado
  return rls.select().from(schema.leads)
}
```

---

## Schema por Módulo

```ts
// db/schema/crm.ts
import { pgTable, uuid, text, timestamp, integer } from 'drizzle-orm/pg-core'
import { pgPolicy } from 'drizzle-orm/pg-core'
import { sql } from 'drizzle-orm'

export const leads = pgTable('leads', {
  id: uuid('id').defaultRandom().primaryKey(),
  internalCode: integer('internal_code').generatedAlwaysAsIdentity(), // identity > serial
  nome: text('nome').notNull(),
  email: text('email'),
  telefone: text('telefone'),
  status: text('status').notNull().default('prospecto'),
  airtableId: text('airtable_id').unique(), // FK para Airtable
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}).withRLS() // .enableRLS() está deprecated desde v1.0-beta.1

export const leadsPolicy = pgPolicy('leads_auth_policy', {
  for: 'all',
  to: 'authenticated',
  using: sql`true`, // ajustar para RBAC real: auth.jwt() ->> 'role' IN ('admin', 'gestor')
})
```

---

## Migrations

```bash
# Gerar SQL a partir do schema
npx drizzle-kit generate

# Aplicar em produção (SEMPRE via DIRECT_URL)
npx drizzle-kit migrate

# NUNCA em produção:
# npx drizzle-kit push  ← só para dev local
```

**CI/CD — aplicar migration no deploy:**
```yaml
# .github/workflows/production.yaml (adicionar antes do deploy Vercel)
- name: Apply Drizzle migrations
  run: npx drizzle-kit migrate
  env:
    DIRECT_URL: ${{ secrets.SUPABASE_DIRECT_URL }}
```

---

## Transactions e Isolation

```ts
// Transaction básica
const result = await db.transaction(async (tx) => {
  const [lead] = await tx.insert(leads).values(data).returning()
  await tx.insert(auditLog).values({ action: 'lead_created', refId: lead.id })
  return lead
})

// Financial operations: usar serializable
await db.transaction(
  async (tx) => { /* operações financeiras */ },
  { isolationLevel: 'serializable', accessMode: 'read write' }
)
```

---

## Type Safety com drizzle-zod

```ts
import { createInsertSchema, createSelectSchema } from 'drizzle-zod'
import { z } from 'zod'

export const insertLeadSchema = createInsertSchema(leads, {
  email: (s) => s.email().optional(),
  telefone: z.string().regex(/^\+?[\d\s-]{8,}$/).optional(),
})

export type Lead = typeof leads.$inferSelect
export type NewLead = typeof leads.$inferInsert
export type InsertLead = z.infer<typeof insertLeadSchema>
```

> ⚠️ Bug conhecido em drizzle-zod@0.6.1: `createInsertSchema` pode retornar `unknown`. Verificar versão instalada antes de usar.

---

## Fontes Oficiais
- https://orm.drizzle.team/docs/connect-supabase
- https://supabase.com/docs/guides/database/connecting-to-postgres
- https://orm.drizzle.team/docs/rls
- https://github.com/rphlmr/drizzle-supabase-rls
- https://orm.drizzle.team/docs/transactions
