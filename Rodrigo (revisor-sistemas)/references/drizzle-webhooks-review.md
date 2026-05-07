# Drizzle ORM + Webhooks — Guia de Revisão

## Drizzle ORM — Bugs Silenciosos

### `.where()` encadeado sobrescreve

```ts
// 🔴 BLOQUEANTE — cada .where() substitui o anterior
const result = await db.select().from(leads)
  .where(eq(leads.status, 'ativo'))       // ignorado
  .where(eq(leads.userId, userId))        // só este aplica
// SQL gerado: WHERE user_id = $1
// Todos os leads do usuário retornam, independente do status

// ✅ CORRETO — and() para múltiplas condições
const result = await db.select().from(leads)
  .where(and(
    eq(leads.status, 'ativo'),
    eq(leads.userId, userId)
  ))
// SQL gerado: WHERE status = $1 AND user_id = $2
```

### `eq(col, null)` gera SQL inválido

```ts
// 🔴 BLOQUEANTE — SQL: WHERE deleted_at = NULL
// Em SQL, NULL = NULL é sempre FALSE → zero resultados, sem erro
const ativos = await db.select().from(leads)
  .where(eq(leads.deletedAt, null)) // silenciosamente retorna []

// ✅ CORRETO
const ativos = await db.select().from(leads)
  .where(isNull(leads.deletedAt))   // SQL: WHERE deleted_at IS NULL
```

### SELECT sem colunas em tabelas com dados sensíveis

```ts
// 🟡 OBRIGATÓRIO — retorna TODAS as colunas incluindo hash de senha, tokens, PII
const users = await db.select().from(usersTable)

// ✅ CORRETO — selecionar apenas o necessário
const users = await db.select({
  id: usersTable.id,
  nome: usersTable.nome,
  email: usersTable.email,
  role: usersTable.role,
  // hash de senha, reset_token, etc. ficam de fora
}).from(usersTable)
```

### `prepare: false` obrigatório com pooler Transaction (porta 6543)

```ts
// 🔴 BLOQUEANTE — omitir prepare causa crash em produção
// Supavisor (Transaction mode) troca a conexão Postgres entre transactions
// Prepared statements são session-scoped → não existem na nova conexão
// Erro: "prepared statement 'drizzle_s1' does not exist"

import postgres from 'postgres'
import { drizzle } from 'drizzle-orm/postgres-js'

// 🔴 ERRADO
const client = postgres(process.env.DATABASE_URL!) // prepare: true por padrão
const db = drizzle(client)

// ✅ CORRETO para pooler Transaction (porta 6543)
const client = postgres(process.env.DATABASE_URL!, { prepare: false })
const db = drizzle(client)

// Para DIRECT_URL (porta 5432) — migrations e scripts
const migrationClient = postgres(process.env.DIRECT_URL!) // prepare: true OK
```

### Drizzle migrations via DIRECT_URL

```ts
// 🔴 BLOQUEANTE — nunca rodar migrations pelo pooler
// drizzle.config.ts
export default defineConfig({
  schema: './src/db/schema',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DIRECT_URL!, // porta 5432 direta — SEMPRE
  },
})
```

### `.returning()` ausente quando resultado é necessário

```ts
// 🟡 OBRIGATÓRIO — insert sem .returning() quando ID é necessário
await db.insert(leads).values(newLead)
// onde está o ID gerado?

// ✅ CORRETO
const [created] = await db.insert(leads).values(newLead).returning()
return created // tem id, createdAt, etc.
```

### Checklist Drizzle por PR

- [ ] `.where()` usa `and()` / `or()` para múltiplas condições? (não encadeado)
- [ ] Comparações com null usam `isNull()` / `isNotNull()`?
- [ ] `db.select()` em tabelas com dados sensíveis especifica colunas?
- [ ] Conexão pooler (6543) tem `prepare: false`?
- [ ] Migrations usam `DIRECT_URL` (5432)?
- [ ] `.withRLS()` nas tabelas novas (não `.enableRLS()`)?
- [ ] `.returning()` quando o resultado é usado?

---

## Webhooks — Guia de Revisão de Segurança

### req.text() ANTES de parsear — regra absoluta

```ts
// 🔴 BLOQUEANTE — stream consumido antes da verificação HMAC
export async function POST(req: Request) {
  const body = await req.json()              // stream consumido aqui
  const rawBody = JSON.stringify(body)       // NÃO é o body original
  const sig = req.headers.get('x-signature')
  verifyHmac(rawBody, sig, secret)           // HMAC sobre bytes diferentes dos originais
  // atacante pode modificar payload e HMAC nunca vai detectar corretamente
}

// ✅ CORRETO
export async function POST(req: Request) {
  const rawBody = await req.text()           // bytes originais preservados
  // verificar HMAC ANTES de qualquer outra coisa
  const sig = req.headers.get('x-signature') ?? ''
  if (!verifyHmac(rawBody, sig, process.env.WEBHOOK_SECRET!)) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }
  const payload = JSON.parse(rawBody)        // só parsear após verificação
}
```

### timingSafeEqual com length check obrigatório

```ts
import { timingSafeEqual } from 'node:crypto'

// 🔴 BLOQUEANTE — === vaza informação via tempo de resposta
if (provided === expected) { ... }

// 🔴 BLOQUEANTE — timingSafeEqual sem checar length
// Lança RangeError se tamanhos diferentes
timingSafeEqual(Buffer.from(provided), Buffer.from(expected))

// ✅ CORRETO
function safeCompare(provided: string, expected: string): boolean {
  const a = Buffer.from(provided)
  const b = Buffer.from(expected)
  return a.length === b.length && timingSafeEqual(a, b)
}

// Para HMAC-SHA256 (GitHub, Stripe, etc.)
import { createHmac } from 'node:crypto'

function verifyHmac(rawBody: string, signature: string, secret: string): boolean {
  const expected = 'sha256=' + createHmac('sha256', secret).update(rawBody).digest('hex')
  const a = Buffer.from(signature)
  const b = Buffer.from(expected)
  return a.length === b.length && timingSafeEqual(a, b)
}

// Para Airtable (shared secret, não HMAC)
function verifyAirtableSecret(req: Request, secret: string): boolean {
  const provided = Buffer.from(req.headers.get('x-airtable-client-secret') ?? '')
  const expected = Buffer.from(secret)
  return provided.length === expected.length && timingSafeEqual(provided, expected)
}
```

### Replay Protection — janela de 5 minutos

```ts
// 🟡 OBRIGATÓRIO para providers que enviam timestamp
const timestamp = req.headers.get('x-timestamp')
if (!timestamp) return Response.json({ error: 'Missing timestamp' }, { status: 400 })

const ageMs = Math.abs(Date.now() - parseInt(timestamp) * 1000)
if (ageMs > 5 * 60 * 1000) {
  return Response.json({ error: 'Stale request' }, { status: 400 })
}

// Alguns providers assinam sobre timestamp + body (ex: Stripe)
const signedPayload = `${timestamp}.${rawBody}`
const expected = createHmac('sha256', secret).update(signedPayload).digest('hex')
```

### Idempotência — webhook_events

```ts
// 🟡 OBRIGATÓRIO — sem idempotência, retries duplicam processamento

// Schema (migration necessária):
// CREATE TABLE webhook_events (
//   id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
//   provider TEXT NOT NULL,
//   provider_event_id TEXT NOT NULL,
//   processed_at TIMESTAMPTZ DEFAULT NOW(),
//   UNIQUE(provider, provider_event_id)
// );

// No handler:
const eventId = payload.id ?? req.headers.get('x-event-id') ?? ''

// INSERT ... ON CONFLICT → retorna 0 rows se já existia
const inserted = await db.insert(webhookEvents)
  .values({ provider: 'airtable', providerEventId: eventId })
  .onConflictDoNothing()
  .returning()

if (inserted.length === 0) {
  return Response.json({ status: 'duplicate' }) // 200, não 4xx
}

// Processar após confirmar que não é duplicata
waitUntil(processEvent(payload))
return Response.json({ received: true })
```

### waitUntil() — processamento assíncrono

```ts
import { waitUntil } from '@vercel/functions'

// 🟡 OBRIGATÓRIO — sem waitUntil(), Vercel encerra a função após o return
// processamento assíncrono é silenciosamente cancelado

// 🔴 ERRADO
export async function POST(req: Request) {
  // ...verificações...
  processHeavyJob(payload) // fire-and-forget — vai ser cancelado
  return Response.json({ received: true })
}

// ✅ CORRETO
export async function POST(req: Request) {
  // ...verificações...
  waitUntil(processHeavyJob(payload)) // Vercel aguarda antes de encerrar
  return Response.json({ received: true }) // responde imediato ao provider
}
```

### Checklist Webhooks por PR

- [ ] `req.text()` é o PRIMEIRO await do handler (antes de qualquer parse)?
- [ ] HMAC/secret verificado com `timingSafeEqual` + length check?
- [ ] Timestamp check (se provider envia) com janela de 5 min?
- [ ] `webhook_events` com `UNIQUE(provider, provider_event_id)` para idempotência?
- [ ] Resposta imediata 200 com `waitUntil()` para processamento pesado?
- [ ] Secrets via env var, não hardcoded?

---

## Fontes
- https://orm.drizzle.team/docs/select
- https://orm.drizzle.team/docs/rls
- https://vercel.com/docs/functions/functions-api-reference#waituntil
- https://hookdeck.com/webhooks/guides/how-to-implement-sha256-webhook-signature-verification
- https://nodejs.org/api/crypto.html#cryptotimingsafeequala-b
