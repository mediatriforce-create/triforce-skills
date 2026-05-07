# Constraints da Plataforma — Referência Completa

> Fonte original: research/dev-web/constraints-plataforma.md (2026-04-13)
> Validado em: research/dev-web/validacao-estrategica.md

---

## Vercel

### Limites por Tier — Tabela Completa

| Constraint | Hobby (Free) | Pro ($20/mês) |
|-----------|-------------|---------------|
| Bandwidth | 100 GB/mês | 1 TB/mês |
| Function Invocations | 1 milhão/mês | 1 milhão incluído + pay-as-you-go |
| Build Execution | 6.000 min/mês | 24.000 min/mês |
| Deployments/dia | 100 | 6.000 |
| Deployments/hora | 100 (rate limit) | sem limite |
| **Concurrent Builds** | **1** | 12 |
| Build time por deployment | 45 min | 45 min |
| Static File upload | 100 MB | 1 GB |
| Serverless duração padrão | 10s | 10s |
| **Serverless duração máxima** | **60s** | 300s (5 min) |
| Edge bundle (após gzip) | **1 MB** | 2 MB |
| Edge duration | 25s para iniciar response | 25s |
| Edge streaming | 300s | 300s |
| **Runtime Logs** | **1 hora** | 1 dia |
| Projects | 200 | ilimitado |
| **Git org connect** | **Não permitido** | Permitido |
| Web Analytics Events | 100.000/mês | 1 milhão/mês |
| Speed Insights Data Points | 10.000/mês | 100.000/mês |

### Análise de Impacto para a Agência

**1 concurrent build (Hobby)** — o gargalo mais frequente. Com 3+ projetos ativos em desenvolvimento simultâneo, builds ficam na fila. Sintoma: "build pendente" por minutos no dashboard Vercel.

**Runtime logs por 1 hora** — torna debug de erros de produção difícil sem Sentry. A integração Sentry (free até 5k erros/mês) compensa este gap completamente.

**60s de função máxima** — LPs simples não chegam perto. Risco real: processamento de formulários pesados (geração de PDF, envio de e-mail batch, webhooks lentos). Solução: Supabase Edge Functions para tarefas longas.

**Git org connect bloqueado** — se o fundador usa uma organização GitHub (ex: `triforce-auto/`), precisa de Pro ou usar repositórios pessoais. Alternativa: transferir repos para conta pessoal.

### Quando Fazer Upgrade para Pro

Critérios objetivos:
- 3+ clientes com projetos em desenvolvimento simultâneo (concurrent builds)
- Precisa de logs por mais de 1 hora sem depender de Sentry
- Repositórios em Git organization
- Funções serverless com processamento > 60s

---

## Supabase

### Limites Free Tier — Tabela Completa

| Constraint | Free | Pro ($25/mês por projeto) |
|-----------|------|--------------------------|
| Database size | 500 MB | 8 GB incluído |
| File Storage | 1 GB | 100 GB incluído |
| Database Egress | 5 GB/mês | 250 GB/mês |
| Storage Egress (cached) | 5 GB/mês | 200 GB/mês |
| **Auth MAUs** | **50.000** | 50.000 incluído |
| **Active Projects** | **2** | ilimitado |
| **Inactivity Pausing** | **1 semana** sem acesso | Nunca pausa |
| **Backups / PITR** | **Nenhum** | Diário (PITR pago à parte) |
| Edge Functions | Incluído (limitado) | Incluído |
| Realtime | Básico | SLA garantido |
| Support | Comunidade | Email 24h |
| Supabase CLI mínimo (gen types) | v1.8.1+ | v1.8.1+ |
| supabase-js (JSON type inference) | v2.48.0+ | v2.48.0+ |

### Análise de Impacto para a Agência

**Pausa após 1 semana (Free)** — o risco mais crítico para LPs em produção. LPs de clientes com baixo tráfego (ex: barbearia que não recebe visitas por uma semana) pausam o projeto Supabase. O primeiro request após pausa tem cold start de 5–10 segundos visíveis para o usuário. Soluções:
- Cron job de wake-up ping a cada 3–4 dias (Vercel Cron ou GitHub Actions)
- Supabase Pro ($25/mês)
- Cloudflare Worker como proxy que faz prefetch (edge não tem esse problema)

**2 projetos no Free** — a agência com 3+ clientes precisa de Pro ou projetos distribuídos em contas Supabase diferentes. Custo: $25/mês por projeto adicional.

**Sem backups** — para clientes pagando pela LP, recomendar Pro para ter backup diário. Sem isso, qualquer problema de banco (drop acidental, SQL errado) é irreversível no Free.

### Quando Fazer Upgrade para Pro

Critérios objetivos:
- LP de cliente em produção com tráfego regular (evitar pausa)
- 3+ projetos ativos
- Cliente exige SLA ou backup de dados
- LP tem área de membros com dados sensíveis

---

## Cloudflare Workers

### Limites por Tier — Tabela Completa

| Constraint | Free | Paid ($5/mês) |
|-----------|------|---------------|
| Requests | 100.000/dia | 10 milhões/mês incluído |
| **CPU por request** | **10ms** | 30s padrão, 5 min máximo |
| Memória por isolate | 128 MB | 128 MB |
| Worker bundle (após gzip) | 3 MB | 10 MB |
| Subrequests por invocação | **50** | 10.000 |
| Workers por conta | 100 | 500 |
| Cron Triggers por conta | 5 | 250 |

### KV (Key-Value Store)

| Constraint | Free | Paid |
|-----------|------|------|
| Reads | 100.000/dia | 10 milhões/mês |
| Writes | 1.000/dia (chaves distintas) | 1 milhão/mês |
| Storage | 1 GB por conta | 1 GB incluído |
| Write para mesma chave | 1/segundo | 1/segundo |

### D1 (Banco de Dados SQLite na Edge)

| Constraint | Free | Paid |
|-----------|------|------|
| Databases | 10 | 50.000 |
| Storage por database | 500 MB | 10 GB |
| Storage total | 5 GB | Ilimitado |
| Queries por invocação | 50 | ilimitado |
| Time Travel (PITR) | 7 dias | 30 dias |

### Análise de Impacto para a Agência

**10ms CPU (Free)** — o limite mais severo. Suficiente para: auth token validation, header manipulation, redirects, A/B routing simples. Insuficiente para: parsing de JSON complexo, transformações, qualquer loop sobre arrays grandes. Para Workers com lógica real: $5/mês é obrigatório.

**Nota importante:** O limite de 10ms CPU é específico para **Cloudflare Workers**. O **Vercel Edge Middleware** (Next.js) não tem esse limite — por isso Vercel Edge Middleware é preferível para lógica de middleware na stack atual da agência.

### Quando Fazer Upgrade

- Worker com lógica além de routing/redirect simples
- Volume de requests > 100.000/dia (campanhas de tráfego pago alto)
- Worker chamando múltiplas APIs externas (> 50 subrequests)

---

## Next.js Edge Runtime

### APIs Indisponíveis no Edge Runtime

| API / Módulo | Por que não funciona | Alternativa |
|-------------|---------------------|-------------|
| `fs`, `path` | Sem filesystem no V8 isolate | Leitura via import estático |
| `net`, `http`, `crypto` nativo | APIs Node.js exclusivas | Web Crypto API (disponível no Edge) |
| `child_process` | Sem processos filhos | Função serverless Node.js |
| `require()` | Somente ES Modules | `import` |
| `eval()` | Proibido por segurança | Refatorar lógica |
| `new Function(string)` | Execução dinâmica bloqueada | Refatorar lógica |
| Conexão PostgreSQL direta | Sem suporte TCP | Usar fetch para API routes |
| ISR (unstable_cache padrão) | Não suportado no Edge | Node.js serverless |
| `NextRequest.geo` / `.ip` | **Removidos no Next.js 15** | `geolocation()` / `ipAddress()` de `@vercel/functions` |

### Bundle Size Limits (Edge Runtime)

| Plan | Limite (após gzip) | Inclui |
|------|-------------------|--------|
| Hobby | **1 MB** | código + libs + assets |
| Pro | 2 MB | idem |
| Enterprise | 4 MB | idem |

### Módulos Node.js Compatíveis com Edge

| Módulo | Suporte |
|--------|---------|
| `async_hooks` (AsyncLocalStorage) | Parcial (WinterCG subset) |
| `events` | Completo |
| `buffer` | Completo |
| `assert` | Completo |
| `util` (promisify, callbackify) | Parcial |
| `Buffer` (global) | Disponível |

### Tabela de Breaking Changes — Next.js 15

| O que mudou | Impacto | Ação necessária |
|-------------|---------|----------------|
| `cookies()`, `headers()`, `draftMode()` async | **CRÍTICO** | Adicionar `await` em todos os usos |
| `params`, `searchParams` async | **CRÍTICO** | Adicionar `await` em layouts/pages |
| `fetch` sem cache por padrão | **ALTO** | Adicionar `{ cache: 'force-cache' }` onde necessário |
| `GET` em Route Handlers sem cache | **MÉDIO** | Usar `export const dynamic = 'force-static'` para optar |
| `NextRequest.geo` e `.ip` removidos | **CRÍTICO** | Migrar para `@vercel/functions` |
| `runtime: 'experimental-edge'` removido | **BAIXO** | Mudar para `'edge'` |
| `@next/font` removido | **BAIXO** | Usar `next/font/google` nativo |
| `useFormState` depreciado | **MÉDIO** | Migrar para `useActionState` (React 19) |
| Speed Insights auto-instrumentation removida | **MÉDIO** | Instalar `@vercel/speed-insights` manualmente |

Codemod disponível para migração: `npx @next/codemod@canary upgrade latest`

---

## Supabase-js — Versão Mínima por Feature

| Feature | Versão mínima |
|---------|--------------|
| JSON/JSONB type inference aprimorada | **v2.48.0+** |
| Supabase CLI para gen types | **v1.8.1+** |
| `QueryData<typeof query>` para joins | qualquer v2.x |

---

## Resumo: Quando Fazer Upgrade

| Plataforma | Trigger de Upgrade | Custo |
|-----------|-------------------|-------|
| Vercel Free → Pro | 3º cliente ativo ou org GitHub | $20/mês |
| Supabase Free → Pro | LP em produção com tráfego real | $25/mês por projeto |
| Cloudflare Workers Free → Paid | Lógica além de routing simples | $5/mês |
