# Operacional — Ferramentas e MCPs

> Referência completa de comandos, MCPs e integrações disponíveis para Felipe.

---

## MCPs Ativos (disponíveis agora)

### Supabase MCP — `mcp__claude_ai_Supabase__*`

| Tool | O que faz | Parâmetros principais |
|------|-----------|----------------------|
| `list_projects` | Lista todos os projetos Supabase da org | — |
| `get_project` | Detalhes de um projeto específico | `project_id` |
| `execute_sql` | Executa SQL direto no banco | `project_id`, `query` |
| `apply_migration` | Aplica DDL versionado (cria tabelas, índices) | `project_id`, `name`, `query` |
| `generate_typescript_types` | Gera tipos TypeScript sem precisar de CLI local | `project_id` |
| `deploy_edge_function` | Deploy de Edge Function | `project_id`, `name`, `entrypoint_path` |
| `get_edge_function` | Detalhes de Edge Function existente | `project_id`, `function_slug` |
| `list_edge_functions` | Lista todas as Edge Functions | `project_id` |
| `get_logs` | Logs em tempo real (API, DB, Edge) | `project_id`, `service` |
| `get_advisors` | Performance advisor automático (índices faltando, etc) | `project_id` |
| `list_tables` | Lista tabelas do schema | `project_id`, `schemas` |
| `list_migrations` | Lista migrações aplicadas | `project_id` |
| `create_branch` | Cria branch de desenvolvimento Supabase | `project_id`, `name` |
| `list_branches` | Lista branches do projeto | `project_id` |
| `merge_branch` | Merge de branch para produção | `branch_id` |
| `get_project_url` | URL pública do projeto | `project_id` |
| `get_publishable_keys` | Chaves anon/service_role | `project_id` |
| `pause_project` | Pausa projeto (economiza free tier) | `project_id` |
| `restore_project` | Restaura projeto pausado | `project_id` |
| `search_docs` | Busca na documentação oficial Supabase | `query` |

**Workflow típico com Supabase MCP:**
```
1. list_projects → pegar project_id
2. apply_migration → criar tabela com DDL
3. execute_sql → verificar dados ou aplicar seeds
4. generate_typescript_types → copiar para src/types/database.types.ts
5. get_advisors → checar se há índices faltando após desenvolvimento
```

---

### Cloudflare MCP — `mcp__claude_ai_Cloudflare_Developer_Platform__*`

| Tool | O que faz |
|------|-----------|
| `accounts_list` | Lista contas Cloudflare disponíveis |
| `set_active_account` | Define conta ativa para operações |
| `workers_list` | Lista Workers deployados |
| `workers_get_worker` | Detalhes de um Worker |
| `workers_get_worker_code` | Código-fonte do Worker |
| `kv_namespaces_list` | Lista namespaces KV |
| `kv_namespace_create` | Cria namespace KV |
| `kv_namespace_get` / `update` / `delete` | Gerencia namespace KV |
| `d1_databases_list` | Lista databases D1 |
| `d1_database_create` | Cria database D1 |
| `d1_database_query` | Executa SQL no D1 |
| `r2_buckets_list` | Lista buckets R2 |
| `r2_bucket_create` | Cria bucket R2 |
| `hyperdrive_configs_list` | Lista configs Hyperdrive (proxy para PostgreSQL) |
| `search_cloudflare_documentation` | Busca docs Cloudflare atualizadas |
| `migrate_pages_to_workers_guide` | Guia de migração Pages → Workers |

---

### Figma MCP — `mcp__claude_ai_Figma__*`

| Tool | O que faz | Quando usar |
|------|-----------|------------|
| `get_design_context` | Extrai design de um node Figma e gera código React/Tailwind | Principal — brief visual → componente |
| `get_screenshot` | Captura screenshot de um frame/component | Referência visual rápida |
| `get_metadata` | Metadados do arquivo (nome, versões, componentes) | Inventário de componentes |
| `get_variable_defs` | Extrai variáveis/tokens do Figma | Importar design tokens |
| `search_design_system` | Busca componentes no design system | Encontrar componentes existentes |
| `get_code_connect_suggestions` | Sugere mapeamentos Code Connect | Setup inicial do projeto |
| `whoami` | Verifica usuário autenticado | Debug de permissão |

**Workflow Figma → Código:**
```
1. Receber URL do Figma com nodeId
2. get_design_context(fileKey, nodeId) → retorna código React + screenshot + hints
3. Adaptar ao stack do projeto (tokens existentes, componentes reutilizáveis)
4. NUNCA copiar o código gerado diretamente — sempre adaptar ao contexto do projeto
```

**Parsing de URL Figma:**
- `figma.com/design/:fileKey/:name?node-id=:nodeId` → converter `-` para `:` no nodeId
- `figma.com/board/:fileKey/:name` → FigJam, usar `get_figjam`

---

## MCPs a Instalar (Onboarding)

### Vercel MCP (PRIORITÁRIO)

```bash
# Instalar
claude mcp add --transport http vercel https://mcp.vercel.com

# Autenticar (após instalar)
# Dentro do Claude Code, rodar:
/mcp
# Selecionar Vercel e seguir OAuth
```

**O que o Vercel MCP permite:**
- Criar e gerenciar projetos Vercel
- Ver deployments e seus status
- Inspecionar logs de build
- Buscar documentação Vercel
- Configurar environment variables
- Gerenciar domínios

---

### Context7 MCP (DOCS ATUALIZADAS)

```bash
npx -y @upstash/context7-mcp@latest
# Ou instalar permanentemente:
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest
```

**Para que serve:**
- Injeta documentação atualizada e versionada de bibliotecas diretamente no contexto
- Evita erros de APIs deprecated (Next.js 14 vs 15, Supabase v1 vs v2)
- Funciona para: Next.js, React, Supabase, Tailwind, TypeScript, e centenas de libs

**Como usar:**
- Ao escrever código com uma lib específica, mencionar "use context7" ou a lib pelo nome
- Context7 injeta automaticamente a documentação correta da versão

---

### GitHub MCP

```bash
# Instalar
claude mcp add github --scope user -- npx -y @modelcontextprotocol/server-github

# Configurar token
export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_seu_token_aqui
# Ou adicionar ao .env do projeto
```

**Scopes necessários no PAT (Personal Access Token):**
- `repo` — leitura e escrita em repositórios
- `workflow` — gerenciar GitHub Actions
- `read:org` — se usando repositórios de organização

**O que permite:**
- Criar e revisar PRs
- Ver status de CI/CD
- Gerenciar branches
- Criar arquivos e commits
- Ler issues e comentários

---

## Comandos CLI Essenciais

### Supabase CLI

```bash
# Instalar
npm install -g supabase

# Login
supabase login

# Gerar tipos TypeScript (vinculado ao projeto remoto)
supabase gen types typescript --project-id "$PROJECT_REF" --schema public > src/types/database.types.ts

# Gerar tipos localmente (dev)
supabase gen types typescript --local > src/types/database.types.ts

# Alternativamente via MCP (sem CLI local):
# mcp__claude_ai_Supabase__generate_typescript_types(project_id: "...")
```

### Vercel CLI

```bash
# Instalar
npm install -g vercel

# Login
vercel login

# Deploy para preview
vercel

# Deploy para produção
vercel --prod

# Ver logs em tempo real
vercel logs [deployment-url]

# Vincular projeto existente
vercel link
```

### Sentry Setup

```bash
# Wizard automático (recomendado — configura tudo)
npx @sentry/wizard@latest -i nextjs

# O wizard cria automaticamente:
# - instrumentation-client.ts
# - sentry.server.config.ts
# - sentry.edge.config.ts
# - instrumentation.ts
# - next.config.ts (wrapped com withSentryConfig)
# - app/global-error.tsx
```

**Variáveis de ambiente obrigatórias:**
```bash
SENTRY_DSN=https://xxx@sentry.io/xxx
SENTRY_AUTH_TOKEN=xxx  # para upload de source maps no deploy
SENTRY_ORG=triforce-auto
SENTRY_PROJECT=cliente-nome
```

---

## Scripts Recomendados (package.json)

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit",
    "gen-types": "supabase gen types typescript --project-id \"$PROJECT_REF\" > src/types/database.types.ts",
    "gen-types:local": "supabase gen types typescript --local > src/types/database.types.ts"
  }
}
```

---

## GitHub Actions — TypeGen Automático

```yaml
# .github/workflows/typegen.yml
name: Supabase TypeGen

on:
  schedule:
    - cron: '0 2 * * *'  # todo dia às 2h
  workflow_dispatch:

jobs:
  generate-types:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm install -g supabase
      - run: |
          supabase gen types typescript \
            --project-id "${{ secrets.SUPABASE_PROJECT_REF }}" \
            > src/types/database.types.ts
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
      - name: Commit se houver mudanças
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add src/types/database.types.ts
          git diff --staged --quiet || git commit -m "chore: update Supabase types"
          git push
```

---

## Instalação de Skills Adicionais

Skills identificadas no research que podem ser instaladas conforme necessidade:

```bash
# Core do workflow LP (Tier 1 — instalar primeiro)
npx skills add https://github.com/vercel-labs/react-best-practices --skill react-best-practices
npx skills add https://github.com/vercel-labs/next-best-practices --skill next-best-practices
npx skills add https://github.com/openai/frontend-skill --skill frontend-skill
npx skills add https://github.com/cloudflare/web-perf --skill web-perf

# Deploy + SEO + Performance (Tier 2)
npx skills add https://github.com/openai/vercel-deploy --skill vercel-deploy
npx skills add https://github.com/sanity-io/seo-aeo-best-practices --skill seo-aeo-best-practices
npx skills add https://github.com/vercel-labs/next-cache-components --skill next-cache-components

# Git/CI Workflow (Tier 3)
npx skills add https://github.com/callstackincubator/github --skill github
npx skills add https://github.com/openai/gh-fix-ci --skill gh-fix-ci
npx skills add https://github.com/vercel-labs/web-design-guidelines --skill web-design-guidelines
```
