# Constraints de Plataforma — MCP Servers

**Ultima atualizacao:** 2026-05-06
**Fontes:** MCP Spec 2025-11-25, npm, Vercel docs, Cloudflare docs, Supabase docs

---

## 1. MCP Protocol Spec

| Item | Valor | Fonte |
|------|-------|-------|
| Versao corrente | **2025-11-25** | modelcontextprotocol.io/specification/2025-11-25 |
| Versao anterior | 2025-06-18 | modelcontextprotocol.io/specification/2025-06-18 |
| Mensagens | JSON-RPC 2.0 | Spec |
| Transports | stdio, Streamable HTTP, SSE (legacy + polling SEP-1699) | Spec |
| Auth padrao | OAuth 2.1 com PKCE | Spec + Auth0 blog |
| MCP Apps | SEP-1865 — UI embarcada | Spec 2025-11-25 |
| Lifecycle states | pendingInit > ready > closed | Spec |
| Tool annotations | readOnlyHint, destructiveHint, idempotentHint, openWorldHint | Spec |

### Regras de Lifecycle
- Rejeitar tudo exceto `initialize` antes de `initialized`
- `initialize` NAO pode estar em batch JSON-RPC
- Manter state machine: pendingInit > ready > closed
- Shutdown stdio: close stdin + SIGTERM
- Shutdown HTTP: DELETE session

---

## 2. SDKs

### @modelcontextprotocol/sdk (TypeScript) — PRINCIPAL

| Item | Valor | Fonte |
|------|-------|-------|
| Versao estavel | **1.29.0** | npm (confirmado Socket.dev/Snyk/NewReleases) |
| v2 status | **2.0.0-alpha.2** (sub-pacotes: /server, /client, /node, /express, /hono) | GitHub releases |
| Projetos dependentes | 12.399+ | npm |
| Schema v1.x | **Zod v3** (raw shapes). Standard Schema (Zod v4, Valibot, ArkType) **apenas em v2 alpha** | Issues #925, #555 |
| Runtimes | Node.js, Bun, Deno | Docs |
| Middleware | Express, Hono, Node.js HTTP | GitHub |
| Docs | ts.sdk.modelcontextprotocol.io | Oficial |

### FastMCP (punkpeye/fastmcp)

| Item | Valor | Fonte |
|------|-------|-------|
| Versao npm | **4.0.1** (breaking: OAuthProxy.authorize valida redirect_uri) | npm/JSR |
| JSR | @punkpeye/fastmcp (@glama/fastmcp) | JSR |
| Stars GitHub | 3.100+ | GitHub |
| Projetos dependentes | 331 | npm |
| EdgeFastMCP | Sim — Cloudflare Workers, Deno Deploy | Docs mintlify |
| Zod | Nativo em addTool | Docs |
| Custom routes | server.addRoute() | Docs |
| Docs | punkpeye-fastmcp.mintlify.app | Oficial |

---

## 3. Plataformas de Deploy

### Vercel

| Limite | Free (Hobby) | Pro ($20/mes) | Fonte |
|--------|-------------|---------------|-------|
| Serverless max duration | 60s | 300s (ate 800s config) | Vercel docs |
| Edge runtime response | 25s inicio, stream ate 300s | Mesmo | Vercel docs |
| Bundle size | 250 MB descomprimido | Mesmo | Vercel docs |
| Request/response body | **4.5 MB** | **4.5 MB** | Vercel docs |
| Memory | 2 GB / 1 vCPU | Mesmo | Vercel docs |
| Deployments | 100/dia | Maior | Vercel docs |
| Concurrent builds | 1 | 3 | Vercel docs |

**mcp-handler (antigo @vercel/mcp-adapter, renomeado):**
- Versao: **1.1.0** (`npm i mcp-handler`). @vercel/mcp-adapter@1.0.0 e stub que re-exporta mcp-handler.
- Requer @modelcontextprotocol/sdk >= 1.26.0
- Streamable HTTP com cold start handling
- OAuth 2.1 built-in
- Template oficial: vercel.com/templates/next.js/model-context-protocol-mcp-with-next-js
- Suporta: Next.js, Nuxt, Svelte

**Impacto em MCP servers:**
- Body de 4.5 MB limita respostas grandes de tools (ex: Airtable com muitos records)
- Duration de 60s (Hobby) limita tools com processamento longo
- Usar Streamable HTTP (nao SSE legacy) para producao

### Cloudflare Workers

| Limite | Free | Paid ($5/mes) | Fonte |
|--------|------|---------------|-------|
| CPU por request | **10ms** | 30s | CF docs |
| Requests/dia | 100.000 | 10M+ | CF docs |
| Subrequests por invocacao | 50 | 1.000 | CF docs |
| Body size | 100 MB | 100 MB | CF docs |
| Durable Objects | Nao | Sim | CF docs |
| Workers KV reads | 100.000/dia | 10M/mes | CF docs |

**Patterns de MCP em Cloudflare:**
- `createMcpHandler()` — stateless, sem Durable Objects
- `McpAgent` — stateful com Durable Objects (requer plano pago)
- Raw `WebStandardStreamableHTTPServerTransport` — controle total
- Auth: Cloudflare Access, GitHub OAuth, Auth0, Stytch, WorkOS

**Impacto em MCP servers:**
- 10ms CPU (free) = suficiente APENAS para routing/proxy simples
- Tools com logica complexa requerem plano pago
- Durable Objects obrigatoria para sessions stateful (McpAgent)

### Supabase Edge Functions

| Limite | Free | Pro ($25/mes) | Fonte |
|--------|------|---------------|-------|
| Runtime | Deno 2.1 | Deno 2.1 | Supabase docs |
| Timeout | 150s | 400s | Supabase docs |
| Memory | 256 MB | 256 MB | Supabase docs |
| Invocacoes/mes | 500.000 | 2M | Supabase docs |
| Funcoes | 25 | 100 | Supabase docs |
| SUPABASE_DB_URL | Auto-injetado (porta 5432 direta) | Mesmo | Supabase docs |

**Opcoes de MCP em Supabase Edge:**
- SDK oficial + WebStandardStreamableHTTPServerTransport + Hono
- mcp-lite (zero-dependency, Zod, Hono) — mais leve
- @rodriguespn/supabase-mcp-handler — turnkey com createEdgeMCPServer()

**Deploy:** `supabase functions deploy --no-verify-jwt mcp`
**Endpoint:** `https://<ref>.supabase.co/functions/v1/mcp`

**Impacto em MCP servers:**
- Deno 2.1 = imports via `npm:pacote` (nao require)
- Nao Node.js — cuidado com dependencias que usam APIs nativas Node
- Auto-injeta SUPABASE_DB_URL — acesso direto ao banco sem config extra

---

## 4. APIs Target (Rate Limits)

| API | Rate Limit | Cooldown | Auth |
|-----|-----------|----------|------|
| Airtable | **5 req/s por base** | 30s em 429 | Bearer token |
| Hotmart | **500 req/min** (leitura + escrita) | -- | OAuth 2.0 client_credentials |
| WhatsApp Cloud API | ~80 msgs/s por numero | -- | Bearer token permanente (System User) |
| Supabase (via MCP oficial) | -- | -- | OAuth 2.0 / JWT |
| Cloudflare (via MCP oficial) | -- | -- | API token |

---

## 5. MCP Servers Pre-Construidos (Stack Triforce)

| Server | Status | Uso |
|--------|--------|-----|
| Supabase MCP (oficial) | **JA INSTALADO** | SQL, schemas, edge functions, auth |
| Airtable MCP (oficial) | **JA DISPONIVEL** | Records, tables, bases |
| Cloudflare MCP (oficial) | **JA INSTALADO** | Workers, KV, D1, R2 |
| Hotmart MCP | **NAO EXISTE** — construir do zero | Sales, subscriptions, webhooks |
| WhatsApp Cloud API MCP | **NAO EXISTE** — construir do zero | Messages, templates, media |
| Airtable customizado (rate limiter) | **NAO EXISTE** — construir sobre o oficial | Rate limiting + DLQ + idempotency |
