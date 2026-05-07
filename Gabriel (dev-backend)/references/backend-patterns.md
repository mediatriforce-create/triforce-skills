# Backend Patterns — Next.js 15 + Supabase + Drizzle

## next-safe-action — Setup Completo

```bash
npm install next-safe-action zod
```

```ts
// lib/safe-action.ts
import { createSafeActionClient } from "next-safe-action"
import { z } from "zod"

// Client base
export const actionClient = createSafeActionClient({
  defaultValidationErrorsShape: "flattened",
})

// Client autenticado
export const authedActionClient = actionClient.use(async ({ next, ctx }) => {
  const supabase = await createClient()
  const { data: { user }, error } = await supabase.auth.getUser()
  if (error || !user) throw new Error("Unauthorized")
  return next({ ctx: { user, supabase } })
})

// Client com rate limiting
export const rateLimitedActionClient = authedActionClient.use(async ({ next, ctx }) => {
  const { success } = await ratelimit.limit(`user_${ctx.user.id}`)
  if (!success) throw new Error("Rate limit exceeded")
  return next({ ctx })
})
```

```ts
// Uso em action
"use server"
import { authedActionClient } from "@/lib/safe-action"

export const createLeadAction = authedActionClient
  .schema(z.object({
    nome: z.string().min(1, "Nome obrigatório"),
    telefone: z.string().regex(/^\+?[\d\s-]{8,}$/, "Telefone inválido"),
    email: z.string().email().optional(),
  }))
  .action(async ({ parsedInput, ctx }) => {
    const [lead] = await db.insert(leads)
      .values({ ...parsedInput, createdBy: ctx.user.id })
      .returning()
    return lead
  })
```

```ts
// Uso no Client Component
"use client"
import { useAction } from "next-safe-action/hooks"
import { createLeadAction } from "@/app/actions/leads"

function CreateLeadForm() {
  const { execute, result, status } = useAction(createLeadAction)

  return (
    <form onSubmit={(e) => {
      e.preventDefault()
      execute({ nome: "...", telefone: "..." })
    }}>
      {result.validationErrors?.nome && (
        <span>{result.validationErrors.nome[0]}</span>
      )}
    </form>
  )
}
```

Docs: https://next-safe-action.dev

---

## Result Pattern (alternativa sem biblioteca)

Para funções internas que não são Server Actions:

```ts
// lib/result.ts
export type Ok<T> = { success: true; data: T }
export type Err<E = AppError> = { success: false; error: E }
export type Result<T, E = AppError> = Ok<T> | Err<E>

export const ok = <T>(data: T): Ok<T> => ({ success: true, data })
export const err = <E>(error: E): Err<E> => ({ success: false, error })

export type AppError =
  | { code: "VALIDATION_ERROR"; fieldErrors: Record<string, string[]> }
  | { code: "NOT_FOUND"; message: string }
  | { code: "UNAUTHORIZED"; message: string }
  | { code: "CONFLICT"; message: string }
  | { code: "INTERNAL_ERROR"; message: string }
```

---

## Webhook Idempotência com Drizzle

```sql
-- Migration: criar tabela de eventos processados
CREATE TABLE webhook_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider_event_id TEXT NOT NULL,
  provider TEXT NOT NULL,
  processed_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(provider, provider_event_id)
);
```

```ts
// app/api/webhooks/[provider]/route.ts
import { waitUntil } from "@vercel/functions"

export async function POST(req: Request) {
  const rawBody = await req.text()
  // ... verificar assinatura ...

  const payload = JSON.parse(rawBody)
  const eventId = payload.id ?? req.headers.get("x-event-id") ?? ""

  // Checar idempotência ANTES de processar
  const existing = await db.select()
    .from(webhookEvents)
    .where(and(
      eq(webhookEvents.provider, "airtable"),
      eq(webhookEvents.providerEventId, eventId)
    ))
    .limit(1)

  if (existing.length > 0) {
    return Response.json({ status: "duplicate" }) // 200, não 4xx
  }

  // Registrar ANTES de processar (garante idempotência mesmo se processar falhar)
  await db.insert(webhookEvents).values({
    provider: "airtable",
    providerEventId: eventId,
  })

  // Processar de forma assíncrona — responder imediato
  waitUntil(processWebhookEvent(payload))
  return Response.json({ received: true })
}
```

---

## Zod Schemas — Padrões

```ts
// db/schema/crm.ts — single source of truth
export const leads = pgTable("leads", {
  id: uuid("id").defaultRandom().primaryKey(),
  nome: text("nome").notNull(),
  email: text("email"),
  telefone: text("telefone"),
  status: text("status").notNull().default("prospecto"),
})

// Schemas derivados do Drizzle (sem duplicação)
import { createInsertSchema, createSelectSchema } from "drizzle-zod"

export const insertLeadSchema = createInsertSchema(leads, {
  email: (s) => s.email().optional(),
  telefone: z.string().regex(/^\+?[\d\s-]{8,}$/).optional(),
})

export const selectLeadSchema = createSelectSchema(leads)
export const updateLeadSchema = insertLeadSchema.partial().omit({ id: true })

export type Lead = typeof leads.$inferSelect
export type NewLead = z.infer<typeof insertLeadSchema>
```

---

## Testing Setup — PGlite

```bash
npm install -D @electric-sql/pglite vitest @vitejs/plugin-react
```

```ts
// vitest.config.ts
import { defineConfig } from "vitest/config"

export default defineConfig({
  test: {
    setupFiles: ["./test/setup.ts"],
    pool: "forks", // obrigatório para PGlite WASM
  },
})
```

```ts
// test/setup.ts
import { PGlite } from "@electric-sql/pglite"
import { drizzle } from "drizzle-orm/pglite"
import * as schema from "@/db/schema"
import { sql } from "drizzle-orm"
import { beforeAll, afterAll, beforeEach } from "vitest"

let client: PGlite
export let testDb: ReturnType<typeof drizzle>

beforeAll(async () => {
  client = new PGlite()
  testDb = drizzle(client, { schema })
  // Criar tabelas (usar SQL das migrations geradas pelo drizzle-kit)
  await testDb.execute(sql`CREATE TABLE IF NOT EXISTS leads (...)`)
})

beforeEach(async () => {
  // Limpar entre testes
  await testDb.execute(sql`TRUNCATE TABLE leads RESTART IDENTITY CASCADE`)
})

afterAll(async () => {
  await client.close()
})
```

Referência: https://github.com/rphlmr/drizzle-vitest-pg

---

## GitHub Actions — Test Pipeline com Supabase Local

```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: supabase/setup-cli@v1
        with:
          version: latest
      - name: Start Supabase
        run: supabase start
      - name: Run migrations
        run: npx drizzle-kit migrate
        env:
          DIRECT_URL: ${{ env.SUPABASE_DB_URL }}
      - name: Run tests
        run: npx vitest run
        env:
          DATABASE_URL: ${{ env.SUPABASE_DB_URL }}
```

> Para testes unitários sem RLS: usar PGlite (mais rápido, sem Docker).
> Para testes de RLS policies: usar `supabase start` + `supabase test db`.

---

## Fontes
- https://next-safe-action.dev
- https://nextjs.org/docs/app/guides/testing/vitest
- https://github.com/rphlmr/drizzle-vitest-pg
- https://supabase.com/docs/guides/local-development/testing/overview
- https://upstash.com/docs/redis/sdks/ratelimit-ts/overview
- https://zod.dev/error-formatting
