# Airtable Integration — Next.js 15 + Supabase

> Base ID atual do CRM Triforce Auto: appDxa9P3sBnuNNc3

## Limites Críticos (Airtable)

| Limite | Valor | Consequência |
|--------|-------|--------------|
| Rate limit por base | **5 req/s** | Queue obrigatória |
| Batch size (create/update) | **10 records/request** | Paginar sempre |
| Cooldown em 429 | **30 segundos** | Backoff ≥ 30s |
| SDK em Edge Runtime | **Não funciona** | Usar fetch raw |
| Latência BR → Airtable (US) | ~150-200ms | Cache obrigatório |

---

## Rate Limiting — Queue Centralizada

```ts
// lib/airtable/queue.ts — token bucket 5 req/s
class AirtableQueue {
  private tokens = 5
  private lastRefill = Date.now()
  private readonly RATE = 5

  async enqueue<T>(fn: () => Promise<T>): Promise<T> {
    const now = Date.now()
    const elapsed = (now - this.lastRefill) / 1000
    this.tokens = Math.min(this.RATE, this.tokens + elapsed * this.RATE)
    this.lastRefill = now

    if (this.tokens < 1) {
      await new Promise(r => setTimeout(r, 1000 / this.RATE))
    }
    this.tokens -= 1
    return fn()
  }
}

export const airtableQueue = new AirtableQueue()
```

---

## Cliente Centralizado

```ts
// lib/airtable/client.ts
import Airtable from 'airtable'
import { airtableQueue } from './queue'

const base = new Airtable({
  apiKey: process.env.AIRTABLE_API_KEY!,
}).base(process.env.AIRTABLE_BASE_ID!) // appDxa9P3sBnuNNc3

// Leitura paginada (SDK Node.js — não edge)
export async function listRecords(tableId: string) {
  return airtableQueue.enqueue(async () => {
    const records: unknown[] = []
    await base(tableId).select({ pageSize: 100 }).eachPage((page, next) => {
      records.push(...page.map(r => ({ id: r.id, ...r.fields })))
      next()
    })
    return records
  })
}

// Edge-compatible (fetch raw)
export async function fetchRecordsEdge(tableId: string) {
  const url = `https://api.airtable.com/v0/${process.env.AIRTABLE_BASE_ID}/${tableId}`
  const res = await fetch(url, {
    headers: { Authorization: `Bearer ${process.env.AIRTABLE_API_KEY}` },
    next: { revalidate: 300, tags: ['airtable', tableId] },
  })
  if (!res.ok) throw new Error(`Airtable error: ${res.status}`)
  return res.json()
}
```

---

## Cache Layer

```ts
// lib/airtable/cache.ts
import { unstable_cache } from 'next/cache'
import { listRecords } from './client'

// IMPORTANTE: Next.js 15 não faz cache por padrão — opt-in obrigatório
export const getCachedLeads = unstable_cache(
  async () => listRecords('Leads'),
  ['airtable-leads'],
  { revalidate: 120, tags: ['airtable', 'airtable-leads'] }
)

// Invalidar após mutação
// import { revalidateTag } from 'next/cache'
// revalidateTag('airtable-leads')
```

---

## Estratégias de Sync

### Opção 1 — Supabase FDW (leitura ad-hoc, zero código)

```sql
-- Executar via Supabase MCP: execute_sql
CREATE EXTENSION IF NOT EXISTS wrappers;

CREATE FOREIGN DATA WRAPPER airtable_wrapper
  HANDLER airtable_fdw_handler
  VALIDATOR airtable_fdw_validator;

CREATE SERVER airtable_server
  FOREIGN DATA WRAPPER airtable_wrapper
  OPTIONS (api_key 'YOUR_API_KEY');

CREATE FOREIGN TABLE airtable_leads (
  id text,
  nome text,
  status text,
  telefone text
) SERVER airtable_server
OPTIONS (base_id 'appDxa9P3sBnuNNc3', table_id 'tblXXXXX');

-- Query como tabela normal + join com Supabase
SELECT l.*, c.email
FROM airtable_leads l
JOIN clientes c ON c.airtable_id = l.id;
```

**Use quando:** leitura ad-hoc, joins ocasionais, sem necessidade de persistir.

### Opção 2 — Webhook Sync (event-driven)

```ts
// app/api/webhooks/airtable/route.ts
import { createClient } from '@/lib/supabase/server'
import { revalidateTag } from 'next/cache'
import { NextRequest } from 'next/server'

export async function POST(request: NextRequest) {
  // Verificar secret compartilhado
  const secret = request.headers.get('x-webhook-secret')
  if (secret !== process.env.AIRTABLE_WEBHOOK_SECRET) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { record, action } = await request.json()
  const supabase = await createClient()

  if (action === 'create' || action === 'update') {
    await supabase.from('leads').upsert({
      airtable_id: record.id,
      nome: record.fields.Nome,
      status: record.fields.Status,
      telefone: record.fields.Telefone,
      updated_at: new Date().toISOString(),
    }, { onConflict: 'airtable_id' })
  }

  // Invalidar cache após sync
  revalidateTag('airtable-leads')
  return Response.json({ ok: true })
}
```

**Configurar no Airtable:** Automations → Trigger: "Record created/updated" → Action: "Send webhook" → URL: `https://seuapp.vercel.app/api/webhooks/airtable`

**Use quando:** dados precisam persistir no Supabase para performance ou disponibilidade.

---

## Estrutura de Arquivos Recomendada

```
lib/airtable/
  client.ts       ← SDK (Node) + fetch raw (Edge) + instância base
  cache.ts        ← unstable_cache wrappers com tags
  queue.ts        ← token bucket 5 req/s
  leads.ts        ← fetchers tipados para tabela Leads
  clientes.ts     ← fetchers tipados para tabela Clientes

app/api/
  airtable/
    leads/route.ts      ← GET: cached read (runtime: edge)
  webhooks/
    airtable/route.ts   ← POST: recebe eventos → upsert Supabase
```

---

## Fontes
- https://support.airtable.com/docs/managing-api-call-limits-in-airtable
- https://supabase.com/docs/guides/database/extensions/wrappers/airtable
- https://support.airtable.com/docs/when-webhook-received-trigger
- https://nextjs.org/docs/app/api-reference/functions/unstable_cache
