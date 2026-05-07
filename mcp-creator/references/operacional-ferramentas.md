# Operacional — Ferramentas e Patterns

**Ultima atualizacao:** 2026-05-06

---

## 1. SDK Oficial (@modelcontextprotocol/sdk)

### Setup basico

```bash
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D typescript @types/node
```

### Tool Registration com Zod

```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { z } from 'zod';

const server = new McpServer({ name: 'triforce-mcp', version: '1.0.0' });

const InputSchema = z.object({
  query: z.string().min(1).max(10_000).describe('Search query'),
  limit: z.number().int().min(1).max(100).default(10).describe('Max results'),
});

server.registerTool('search', {
  title: 'Search Records',
  description: 'Search records in the database',
  inputSchema: InputSchema,
  annotations: { readOnlyHint: true, openWorldHint: false },
}, async (input) => {
  const parsed = InputSchema.parse(input);
  const results = await apiClient.search(parsed.query, parsed.limit);
  return {
    content: [{ type: 'text', text: JSON.stringify(results) }],
  };
});
```

### Transports

```typescript
// stdio (local/CLI)
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
const transport = new StdioServerTransport();
await server.connect(transport);

// Streamable HTTP (producao)
import { StreamableHTTPServerTransport } from '@modelcontextprotocol/sdk/server/streamableHttp.js';
// Integrar com Express, Hono ou Node.js HTTP nativo
```

### Capabilities e listChanged

```typescript
const server = new McpServer({
  name: 'triforce-airtable',
  version: '2.0.0',
  capabilities: {
    tools: { listChanged: true } // permite deprecar tools em runtime
  }
});

// Deprecar tool em runtime:
server.removeTool('search_records');
// SDK emite notifications/tools/list_changed automaticamente
```

---

## 2. FastMCP (punkpeye/fastmcp)

### Setup basico

```bash
npm install fastmcp zod
```

### Tool com Zod nativo

```typescript
import { FastMCP } from 'fastmcp';
import { z } from 'zod';

const server = new FastMCP({ name: 'triforce-hotmart', version: '1.0.0' });

server.addTool({
  name: 'hotmart_get_sales_history',
  description: 'Get sales history with filters. Returns paginated results.',
  parameters: z.object({
    startDate: z.string().describe('Start date (YYYY-MM-DD)'),
    endDate: z.string().describe('End date (YYYY-MM-DD)'),
    status: z.enum(['APPROVED', 'REFUNDED', 'CANCELED']).optional().describe('Filter by status'),
    limit: z.number().int().min(1).max(100).default(50).describe('Page size'),
  }),
  annotations: { readOnlyHint: true },
  execute: async (args) => {
    const results = await hotmartClient.getSalesHistory(args);
    return JSON.stringify(results);
  },
});
```

### EdgeFastMCP para Cloudflare Workers

```typescript
import { EdgeFastMCP } from 'fastmcp/edge';
import { z } from 'zod';

const server = new EdgeFastMCP({ name: 'triforce-edge', version: '1.0.0' });

server.addTool({
  name: 'example_tool',
  description: 'Example tool running on the edge',
  parameters: z.object({ input: z.string() }),
  execute: async (args) => `Processed: ${args.input}`,
});

export default { fetch: server.fetch };
```

### Custom HTTP Routes

```typescript
// Health check
server.addRoute('GET', '/health', async () => {
  return {
    status: 'healthy',
    version: '1.0.0',
    tools_registered: server.getTools().length,
    uptime_seconds: process.uptime(),
  };
});

// Webhook receiver (ex: Hotmart postback)
server.addRoute('POST', '/webhooks/hotmart', async (req) => {
  const body = await req.text();
  // verificar HMAC, processar evento
  return { received: true };
});
```

### Dev mode

```bash
npx fastmcp dev src/server.ts
```

---

## 3. Estrutura Padrao de Projeto

```
project/
  src/
    server.ts          # McpServer/FastMCP instance + tool registration
    tools/
      toolA.ts         # Tool handler + Zod schema
      toolB.ts
    resources/
      resourceA.ts
    prompts/
      promptA.ts
    lib/
      api-client.ts    # HTTP client com retry/rate-limit
      auth.ts          # OAuth 2.1 validation / token refresh
      errors.ts        # Error formatting educativo
      schemas.ts       # Shared Zod schemas
      rate-limiter.ts  # Bottleneck / token bucket
  transports/
    stdio.ts           # Entry point stdio
    http.ts            # Entry point Streamable HTTP
  tests/
    tools.test.ts
    evaluations.xml    # 10 questoes de avaliacao
  CHANGELOG.md
  package.json
  tsconfig.json
```

---

## 4. Rate Limiting (Bottleneck)

### Pattern para Airtable (5 req/s, cooldown 30s em 429)

```typescript
import Bottleneck from 'bottleneck';

const limiter = new Bottleneck({
  minTime: 250,      // 1 request a cada 250ms = 4 req/s (buffer)
  maxConcurrent: 1,  // serializar requests
});

// Dead-Letter Queue
interface DeadLetterEntry {
  idempotencyKey: string;
  error: string;
  payload?: unknown;
  timestamp: number;
  retryCount: number;
}
const deadLetterQueue: DeadLetterEntry[] = [];

// Retry com exponential backoff
limiter.on('failed', async (error: any, jobInfo) => {
  const maxRetries = 3;
  if (jobInfo.retryCount < maxRetries) {
    if (error.status === 429) return 30_000; // 30s cooldown Airtable
    return Math.pow(2, jobInfo.retryCount) * 2000 + Math.random() * 1000;
  }
  deadLetterQueue.push({
    idempotencyKey: jobInfo.options.id ?? 'unknown',
    error: error.message || 'Max retries exceeded',
    timestamp: Date.now(),
    retryCount: maxRetries,
  });
  return null;
});

// Idempotency store
const processedKeys = new Map<string, { result: unknown; timestamp: number }>();
const TTL = 24 * 60 * 60 * 1000; // 24h

export async function executeRequest<T>(
  key: string,
  fn: () => Promise<T>,
): Promise<T | { status: 'skipped'; reason: 'idempotency_hit' }> {
  const cached = processedKeys.get(key);
  if (cached && Date.now() - cached.timestamp < TTL) {
    return { status: 'skipped', reason: 'idempotency_hit' };
  }
  return limiter.schedule({ id: key }, async () => {
    const result = await fn();
    processedKeys.set(key, { result, timestamp: Date.now() });
    return result;
  });
}
```

### Consideracoes para deploy distribuido
- **In-memory:** funciona para stdio, single-instance
- **Serverless/Edge:** usar Upstash Redis para idempotency + DLQ
- **Bottleneck v3 (@sderrow/bottleneck):** Redis clustering nativo

---

## 5. Deploy — Comandos por Plataforma

### Vercel (Next.js + mcp-handler)

```typescript
// app/api/mcp/route.ts
import { createMcpHandler } from 'mcp-handler';
import { z } from 'zod';

const handler = createMcpHandler(
  (server) => {
    server.tool('example', { input: z.object({ query: z.string() }) },
      async ({ input }) => ({ content: [{ type: 'text', text: input.query }] })
    );
  },
  { name: 'triforce-mcp', version: '1.0.0' },
  { basePath: '/api/mcp' }
);

export { handler as GET, handler as POST, handler as DELETE };
```

```bash
vercel deploy
```

### Cloudflare Workers

```bash
npm create cloudflare@latest -- my-mcp --template=cloudflare/ai/demos/remote-mcp-authless
wrangler deploy
```

### Supabase Edge Functions

```typescript
// supabase/functions/mcp/index.ts
import { McpServer } from 'npm:@modelcontextprotocol/sdk/server/mcp.js';
import { Hono } from 'npm:hono';

const app = new Hono();
// ... setup MCP server com Hono middleware
Deno.serve(app.fetch);
```

```bash
supabase functions deploy --no-verify-jwt mcp
```

---

## 6. MCP Inspector — Workflow de Teste

```bash
# Iniciar Inspector
npx -y @modelcontextprotocol/inspector

# Conectar ao server local
# URL: stdio://node dist/server.js
# ou: http://localhost:3000/api/mcp

# Verificar:
# 1. Conexao estabelecida (lifecycle OK)
# 2. tools/list retorna todas as tools esperadas
# 3. Cada tool tem inputSchema Zod correto
# 4. Chamar uma tool read-only e verificar response
# 5. Verificar tool annotations
```

---

## 7. Registro no Claude Code

### Projeto local (mcp.json)

```json
{
  "mcpServers": {
    "triforce-hotmart": {
      "type": "stdio",
      "command": "node",
      "args": ["./dist/server.js"],
      "env": {
        "HOTMART_CLIENT_ID": "${HOTMART_CLIENT_ID}",
        "HOTMART_CLIENT_SECRET": "${HOTMART_CLIENT_SECRET}"
      }
    }
  }
}
```

### Global (~/.claude.json)

Editar programaticamente para evitar erros do CLI:

```bash
# Backup + editar
cp ~/.claude.json ~/.claude.json.bak
# Usar jq ou script Python para injetar entrada
```

---

## 8. Integracao Hotmart

### Auth (OAuth 2.0 client_credentials)

```typescript
async function getHotmartToken(): Promise<string> {
  const res = await fetch('https://developers.hotmart.com/security/oauth/token', {
    method: 'POST',
    headers: {
      'Authorization': `Basic ${Buffer.from(`${CLIENT_ID}:${CLIENT_SECRET}`).toString('base64')}`,
      'Content-Type': 'application/json',
    },
    body: 'grant_type=client_credentials',
  });
  const { access_token } = await res.json();
  return access_token; // expira em ~1h, implementar refresh
}
```

### Endpoints principais

| Endpoint | Metodo | Dados |
|----------|--------|-------|
| `/sales/history` | GET | Historico de vendas |
| `/sales/users` | GET | Participantes da venda |
| `/sales/price-details` | GET | Breakdown de preco |
| `/subscriptions/purchases` | GET | Assinaturas |
| `/subscriptions/cancel` | POST | Cancelar assinatura |
| `/club/students` | GET | Alunos matriculados |
| `/club/progress` | GET | Progresso do aluno |

### Rate limit: 500 req/min. Buffer seguro: 8 req/s.

### Webhook Events

| Evento | Descricao |
|--------|-----------|
| `PURCHASE_APPROVED` | Compra aprovada |
| `PURCHASE_REFUNDED` | Reembolso |
| `SUBSCRIPTION_CANCELLATION` | Cancelamento de assinatura |
| `CART_ABANDONMENT` | Carrinho abandonado |

### Tools recomendadas para o MCP

```
hotmart_get_sales_history    (readOnly, paginado)
hotmart_get_sale_details     (readOnly)
hotmart_get_subscriptions    (readOnly)
hotmart_cancel_subscription  (destructive)
hotmart_get_students         (readOnly)
hotmart_get_student_progress (readOnly)
hotmart_get_product_plans    (readOnly)
```

---

## 9. Integracao WhatsApp Cloud API

### Auth (Bearer token permanente)

```typescript
const WA_TOKEN = process.env.WHATSAPP_ACCESS_TOKEN!;
const WA_PHONE_ID = process.env.WHATSAPP_PHONE_NUMBER_ID!;

async function sendMessage(to: string, message: string) {
  return fetch(`https://graph.facebook.com/v21.0/${WA_PHONE_ID}/messages`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${WA_TOKEN}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      messaging_product: 'whatsapp',
      to: to.replace(/\D/g, ''),
      type: 'text',
      text: { body: message },
    }),
  });
}
```

### Webhook (X-Hub-Signature-256)

```typescript
import crypto from 'crypto';

function verifyWebhook(body: string, signature: string): boolean {
  const expected = 'sha256=' + crypto
    .createHmac('sha256', process.env.WHATSAPP_APP_SECRET!)
    .update(body)
    .digest('hex');
  return signature === expected;
}
```

### Messaging Tiers

| Tier | Limite (24h rolling) |
|------|---------------------|
| 0 | ~250 conversas |
| 1 | 1.000 |
| 2 | 10.000 |
| 3 | 100.000 |
| Unlimited | Sem limite |

### Tools recomendadas para o MCP

```
whatsapp_send_text          (nao-destrutivo, idempotente)
whatsapp_send_template      (nao-destrutivo)
whatsapp_send_media         (nao-destrutivo)
whatsapp_send_interactive   (nao-destrutivo)
whatsapp_mark_as_read       (nao-destrutivo, idempotente)
whatsapp_upload_media       (nao-destrutivo)
whatsapp_download_media     (readOnly)
whatsapp_list_templates     (readOnly)
whatsapp_get_business_profile (readOnly)
```

---

## 10. Integracao Airtable Rate Limiter

### Problema
- Airtable: 5 req/s por base, cooldown 30s em 429
- Gabriel pediu: fila + retry + idempotencia + dead-letter encapsulados no MCP

### Solucao: MCP customizado sobre o oficial

```
Camada MCP Airtable Customizado
  |
  +-- Bottleneck (token bucket, 4 req/s buffer)
  |
  +-- Retry (exponential backoff + 30s para 429)
  |
  +-- Idempotency Store (Map in-memory ou Redis)
  |
  +-- Dead-Letter Queue (falhas apos 3 retries)
  |
  +-- Airtable API (oficial)
```

### Tools do MCP customizado

```
airtable_list_records       (readOnly, paginado, rate-limited)
airtable_get_record         (readOnly, rate-limited)
airtable_create_record      (idempotent via key, rate-limited)
airtable_update_record      (idempotent via key, rate-limited)
airtable_delete_record      (destructive, rate-limited)
airtable_batch_create       (idempotent, max 10 por batch)
airtable_batch_update       (idempotent, max 10 por batch)
airtable_get_dead_letter    (readOnly — inspecionar DLQ)
airtable_retry_dead_letter  (retry entry especifica)
airtable_get_rate_status    (readOnly — jobs pendentes, DLQ size)
```

### Para deploy distribuido (Supabase Edge / Cloudflare Workers)
- Usar Upstash Redis para idempotency store e DLQ
- Bottleneck v3 (@sderrow/bottleneck) com Redis clustering
