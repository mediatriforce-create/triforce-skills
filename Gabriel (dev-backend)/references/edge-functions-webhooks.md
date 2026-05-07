# Supabase Edge Functions + Webhooks — Referência

## Edge Functions — Estrutura

```
supabase/
  functions/
    _shared/           ← NÃO deployado (underscore = privado)
      cors.ts
      db.ts            ← Drizzle client factory
      schema.ts        ← re-exporta schema do projeto
    process-lead/
      index.ts
    sync-airtable/
      index.ts
```

```ts
// supabase/functions/_shared/cors.ts
export const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers": "authorization, x-client-info, apikey, content-type",
}

// supabase/functions/_shared/db.ts
import { drizzle } from "npm:drizzle-orm/postgres-js"
import postgres from "npm:postgres"
import * as schema from "../../../src/db/schema/index.ts" // ajustar path

export function createDb() {
  // SUPABASE_DB_URL é auto-injetado (porta 5432 direta)
  const client = postgres(Deno.env.get("SUPABASE_DB_URL")!, { prepare: false })
  return { db: drizzle(client, { schema }), client }
}
```

## Edge Function — Template Completo

```ts
// supabase/functions/minha-funcao/index.ts
import { corsHeaders } from "../_shared/cors.ts"
import { createDb } from "../_shared/db.ts"

Deno.serve(async (req: Request) => {
  // 1. CORS preflight
  if (req.method === "OPTIONS") {
    return new Response("ok", { headers: corsHeaders })
  }

  // 2. Auth: verificar JWT do Supabase
  const authHeader = req.headers.get("Authorization")
  if (!authHeader) {
    return new Response("Unauthorized", { status: 401 })
  }

  // 3. Env vars (NÃO process.env — é Deno)
  const apiKey = Deno.env.get("MY_API_KEY")!

  // 4. DB com Drizzle
  const { db, client } = createDb()

  try {
    const body = await req.json()
    const result = await db.select().from(schema.leads)
    return Response.json(result, { headers: corsHeaders })
  } catch (error) {
    return Response.json({ error: "Internal error" }, { status: 500, headers: corsHeaders })
  } finally {
    // OBRIGATÓRIO — fechar conexão antes de retornar
    await client.end()
  }
})
```

## Diferenças Deno vs Node.js

| Concern | Node.js | Deno 2.1 |
|---------|---------|----------|
| Env vars | `process.env.VAR` | `Deno.env.get("VAR")` |
| HTTP server | Express / `http.createServer` | `Deno.serve()` |
| Imports npm | `import pkg from 'pkg'` | `import pkg from 'npm:pkg'` |
| Std lib | `node:crypto`, `node:fs` | `node:` compat layer OU `Deno.*` |
| Entry point | `export default handler` | `Deno.serve(handler)` |
| Top-level await | Limitado | Nativo |

## Deploy e Secrets

```bash
# Testar local (hot reload)
supabase functions serve minha-funcao --env-file ./supabase/.env.local

# Testar com curl
curl -i --location --request POST \
  'http://127.0.0.1:54321/functions/v1/minha-funcao' \
  --header 'Authorization: Bearer SUPABASE_ANON_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"key":"value"}'

# Configurar secrets em produção
supabase secrets set MY_SECRET=valor
supabase secrets set --env-file .env.production

# Deploy
supabase functions deploy minha-funcao

# Deploy todos
supabase functions deploy
```

## Edge Function vs Next.js API Route — Quando Usar

| Use Edge Function | Use API Route |
|-------------------|---------------|
| Webhook receiver do Airtable (próximo ao DB) | CRUD padrão acoplado ao frontend |
| Background jobs (sync Airtable → Supabase) | Auth callbacks |
| Lógica que precisa ser invocada por Supabase Triggers/Hooks | Processamento com libs Node.js nativas |
| Processamento pesado sem depender de UI | Server Actions |

---

## Webhooks — Padrões de Segurança

### Por que `req.text()` ANTES de parsear

```ts
// ❌ ERRADO — re-serialização muda bytes
const body = await req.json()
const rawBody = JSON.stringify(body) // NÃO é o body original

// ✅ CORRETO — bytes originais para HMAC
const rawBody = await req.text()
// ...verificar HMAC...
const payload = JSON.parse(rawBody)
```

### Por que `timingSafeEqual`

Regular `===` faz curto-circuito na primeira diferença — atacante mede tempo de resposta para força-bruta o secret byte a byte. `timingSafeEqual` sempre demora o mesmo tempo.

```ts
import { timingSafeEqual } from "node:crypto"

// OBRIGATÓRIO: verificar tamanho antes (timingSafeEqual lança se tamanhos diferentes)
const a = Buffer.from(provided)
const b = Buffer.from(expected)
if (a.length !== b.length || !timingSafeEqual(a, b)) {
  return Response.json({ error: "Unauthorized" }, { status: 401 })
}
```

### HMAC-SHA256 (GitHub, Stripe, etc.)

```ts
import { createHmac, timingSafeEqual } from "node:crypto"

function verifyHmacSignature(
  rawBody: string,
  signatureHeader: string,
  secret: string,
  prefix = "sha256=" // alguns providers não têm prefix
): boolean {
  const expected = prefix + createHmac("sha256", secret).update(rawBody).digest("hex")
  const a = Buffer.from(signatureHeader)
  const b = Buffer.from(expected)
  return a.length === b.length && timingSafeEqual(a, b)
}
```

### Em Edge Runtime (sem `node:crypto`)

```ts
// Web Crypto API — funciona em Edge Runtime e Deno
async function verifyHmacWebCrypto(rawBody: string, signature: string, secret: string) {
  const encoder = new TextEncoder()
  const key = await crypto.subtle.importKey(
    "raw", encoder.encode(secret),
    { name: "HMAC", hash: "SHA-256" },
    false, ["sign"]
  )
  const sig = await crypto.subtle.sign("HMAC", key, encoder.encode(rawBody))
  const hex = Array.from(new Uint8Array(sig))
    .map(b => b.toString(16).padStart(2, "0")).join("")
  // Timing-safe compare com strings
  return hex === signature.replace("sha256=", "")
}
```

### Airtable Webhook (shared secret, não HMAC)

```ts
// Airtable envia x-airtable-client-secret com valor fixo (não HMAC)
const provided = Buffer.from(req.headers.get("x-airtable-client-secret") ?? "")
const expected = Buffer.from(process.env.AIRTABLE_WEBHOOK_SECRET!)
if (provided.length !== expected.length || !timingSafeEqual(provided, expected)) {
  return Response.json({ error: "Unauthorized" }, { status: 401 })
}
```

### Replay protection (timestamp)

```ts
// Alguns providers incluem timestamp na assinatura
const timestamp = req.headers.get("x-timestamp")!
const ageMs = Date.now() - parseInt(timestamp) * 1000

// Rejeitar webhooks com mais de 5 minutos
if (ageMs > 5 * 60 * 1000) {
  return Response.json({ error: "Stale request" }, { status: 400 })
}

// Assinar: HMAC(timestamp + "." + rawBody)
const signedPayload = `${timestamp}.${rawBody}`
```

---

## Upstash Rate Limiting — Referência Rápida

```ts
// lib/ratelimit.ts
import { Ratelimit } from "@upstash/ratelimit"
import { Redis } from "@upstash/redis"

// Proteção geral de APIs (por IP)
export const ipRatelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(20, "10 s"),
  analytics: true,
  prefix: "@triforce/ip",
})

// Proteção por usuário (ações custosas)
export const userRatelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, "1 m"),
  analytics: true,
  prefix: "@triforce/user",
})
```

```ts
// middleware.ts — proteção global de /api/*
import { ipAddress } from "@vercel/functions" // NÃO request.ip (deprecated)

export async function middleware(req: NextRequest) {
  if (req.nextUrl.pathname.startsWith("/api/")) {
    const ip = ipAddress(req) ?? "anonymous"
    const { success } = await ipRatelimit.limit(ip)
    if (!success) {
      return new Response("Too Many Requests", { status: 429 })
    }
  }
  return NextResponse.next()
}
```

### Free tier: 10.000 commands/dia
- `slidingWindow` usa ~2 commands/request
- ~5.000 requests/dia no free tier
- $0.2/100k commands no Pay-as-you-go

---

## Fontes
- https://supabase.com/docs/guides/functions
- https://supabase.com/docs/guides/functions/development-environment
- https://orm.drizzle.team/docs/tutorials/drizzle-with-supabase-edge-functions
- https://upstash.com/docs/redis/sdks/ratelimit-ts/overview
- https://hookdeck.com/webhooks/guides/how-to-implement-sha256-webhook-signature-verification
