# Estrategico — Frameworks de Decisao

**Ultima atualizacao:** 2026-05-06

---

## 1. Framework de 4 Fases (Research > Implement > Review > Evaluate)

### Fase 1: Research & Planning
- Entender a API target (endpoints, auth, rate limits, webhooks)
- Buscar specs existentes (OpenAPI, Postman, SDK codegen)
- Verificar se MCP server ja existe (mcpmarket.com, mcp.directory, pulsemcp.com)
- Definir scope: quais tools sao necessarias? Quem vai consumir?
- Output: documento de design com lista de tools + schemas

### Fase 2: Implementation
- Escolher framework: SDK oficial (complexo) ou FastMCP (simples/medio)
- Project structure padrao (src/server.ts, src/tools/, src/lib/)
- Zod em todos os parametros — sem excecao
- Tool annotations em todas as tools
- Rate limiting se API target tem limites
- Output: codigo funcional + testes no Inspector

### Fase 3: Review & Refine
- PR para Andre (dev-lider) — decisoes de arquitetura
- Rodrigo (revisor-sistemas) — seguranca, TypeScript quality
- Luna (luna-qa) — cobertura de testes
- CI verde (lint + typecheck + build)
- Output: PR aprovado e pronto para deploy

### Fase 4: Evaluation
- 10 questoes complexas para validar o MCP server
- Cada questao testa um fluxo real que o agent executaria
- Medir: tool chamada corretamente? Response util? Erros educativos?
- Output: evaluation report com score por questao

---

## 2. Design Agent-Centric

### Principio
Tools existem para o agent resolver tasks do usuario, nao para expor endpoints.

### Regras

| Bom (agent-centric) | Ruim (API wrapper) |
|---------------------|-------------------|
| `get_sales_summary` (agrega dados) | `GET /api/v1/sales` (raw endpoint) |
| `find_customer_by_phone` (busca semantica) | `GET /customers?phone=X` (query param) |
| `cancel_subscription_with_reason` (workflow) | `POST /subscriptions/cancel` (CRUD) |
| `get_daily_revenue_report` (dashboard) | `GET /analytics/revenue?date=X` (raw) |

### Quantas tools?
- **Ate 20 tools:** ideal para agents (Claude, GPT) consumirem diretamente
- **20-50 tools:** funciona mas agents podem confundir. Descriptions precisas sao criticas
- **50+ tools:** usar Code Mode (FastMCP) — colapsa em 3 meta-tools (search, get_schema, execute)

### Descriptions
- Primeira frase: o que a tool faz (verbo no imperativo)
- Segunda frase: quando usar (contexto)
- Listar parametros obrigatorios e opcionais
- Incluir exemplos de valores validos
- Incluir valores de enum completos (nunca truncar)

---

## 3. Tool Annotations Reference

| Annotation | Tipo | Descricao | Exemplo |
|-----------|------|-----------|---------|
| `readOnlyHint` | boolean | Tool nao modifica estado externo | `list_records`, `get_status` |
| `destructiveHint` | boolean | Tool deleta ou modifica irreversivelmente | `delete_record`, `cancel_subscription` |
| `idempotentHint` | boolean | Chamadas repetidas produzem mesmo resultado | `mark_as_read`, `upsert_record` |
| `openWorldHint` | boolean | Tool acessa recursos fora do contexto controlado | `search_web`, `fetch_url` |

### Combinacoes comuns

| Cenario | readOnly | destructive | idempotent | openWorld |
|---------|----------|-------------|------------|-----------|
| Listar records | true | false | true | false |
| Criar record | false | false | true (com idempotency key) | false |
| Deletar record | false | true | true | false |
| Buscar na web | true | false | true | true |
| Enviar email | false | false | false (nao idempotente) | true |

---

## 4. Security Checklist Completo

### Obrigatorio em todo MCP server

- [ ] HTTPS obrigatorio para endpoints remotos (exceto loopback dev)
- [ ] OAuth 2.1 com PKCE para clients publicos
- [ ] Bearer token validation em cada request (issuer, audience, expiry, scopes)
- [ ] MCP-specific scopes (ex: `mcp:read:crm`, `mcp:write:tickets`)
- [ ] Rate limiting por tenant + tool (sliding window ou token bucket)
- [ ] Input validation Zod em TODOS os parametros
- [ ] Bloqueio de IPs privados/reservados (anti-SSRF)
- [ ] Secrets via env vars ou vault (nunca em parametros de tool)
- [ ] Container/edge sandbox com filesystem minimo
- [ ] DNS rebinding protection para servers locais
- [ ] Refresh token rotation para clients publicos
- [ ] Logging de seguranca sem exposicao de dados sensiveis

### Validacao de input (Zod patterns)

```typescript
// Strings: sempre com min/max e describe
z.string().min(1).max(10_000).describe('Search query')

// Numeros: bounds e inteiro quando aplicavel
z.number().int().min(1).max(100).default(10).describe('Page size')

// Enums: valores explicitos
z.enum(['APPROVED', 'REFUNDED', 'CANCELED']).describe('Sale status')

// Emails
z.string().email().describe('Customer email')

// URLs: validar protocolo
z.string().url().refine(u => u.startsWith('https://'), 'HTTPS required')

// IDs: formato especifico
z.string().regex(/^[a-zA-Z0-9_-]+$/).max(128).describe('Record ID')
```

### SSRF Prevention

```typescript
function isPrivateIP(ip: string): boolean {
  return /^(10\.|172\.(1[6-9]|2\d|3[01])\.|192\.168\.|127\.|0\.|169\.254\.|fc|fd)/.test(ip);
}
// Bloquear requests a IPs privados em todas as tools que fazem fetch externo
```

---

## 5. Monitoring Stack (OpenTelemetry)

### Setup basico

```typescript
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';

const sdk = new NodeSDK({
  instrumentations: [getNodeAutoInstrumentations()],
});
sdk.start();
```

### 5 Metricas Obrigatorias

| Metrica | Tipo OTel | Atributos |
|---------|-----------|-----------|
| `mcp.tool.calls` | Counter | `mcp.tool.name`, `mcp.server.name`, `mcp.transport` |
| `mcp.tool.duration` | Histogram | P50, P95, P99 por tool |
| `mcp.tool.errors` | Counter | Erros thrown + texto comecando com "Error:" |
| `mcp.connections.active` | Gauge | Conexoes ativas (HTTP/SSE/stdio) |
| `mcp.requests.queued` | Gauge | Requests na fila (se rate limiting) |

### Alertas Recomendados

| Alerta | Condicao | Acao |
|--------|----------|------|
| Error Rate Alto | `rate(errors) / rate(calls) > 0.05` (5%) | Investigar API downstream |
| Latencia Alta | P95 duration > 1000ms | Verificar throttling |
| Volume Anomalo | Spike em tool.calls | Detectar retry loop infinito |
| DLQ Crescendo | `dlq.size > 10` em 5min | Investigar falhas sistemicas |

### Health Check Endpoint

```typescript
// GET /health
{
  "status": "healthy",
  "version": "1.2.0",
  "uptime_seconds": 3600,
  "tools_registered": 8,
  "active_connections": 3,
  "queue_depth": 0,
  "dlq_size": 0,
  "last_error": null
}
```

---

## 6. Changelog/Versioning Workflow

### Template (Keep a Changelog + Conventional Commits)

```markdown
# Changelog - [nome-do-mcp-server]

## [2.0.0] - 2026-05-06

### Breaking Changes
- **REMOVED** `search_records` tool (use `query_records_v2`)
- **CHANGED** `create_record` now requires `idempotency_key` parameter

### Tool Contracts
- **ADDED** `query_records_v2` — novo tool com paginacao cursor-based
- **DEPRECATED** `search_records` — sera removido em v3.0.0
- **SCHEMA** `update_record`: campo `metadata` agora aceita nested objects

### Protocol
- Atualizado para MCP spec 2025-11-25

### Fixes
- Corrigido timeout em chamadas com payload > 1MB

## [1.2.0] - 2026-04-15

### Added
- **NEW TOOL** `batch_update` — atualizacao em lote com idempotencia
```

### Automacao

| Ferramenta | URL | Uso |
|-----------|-----|-----|
| semantic-release | github.com/semantic-release/semantic-release | Versioning + changelog via Conventional Commits |
| git-cliff | github.com/orhun/git-cliff | Gerador de changelog customizavel |

### Semver para MCP Servers

| Tipo | Quando | Exemplo |
|------|--------|---------|
| **MAJOR** | Breaking changes em tool schemas, remocao de tools, mudanca de auth | `search_records` removido |
| **MINOR** | Novas tools, campos opcionais adicionados | `batch_update` adicionado |
| **PATCH** | Bug fixes, performance, ajustes de descriptions | Timeout corrigido |

### Handling Breaking Changes

| Regra | Descricao |
|-------|-----------|
| Nunca renomear tool existente | Criar novo tool ID (ex: `process_data_v2`) |
| Nunca modificar parametros required in-place | Adicionar novo tool com schema atualizado |
| Test fixtures de regressao | Manter replay de chamadas com schemas anteriores |
| Version negotiation | Permitir clients negociarem versao compativel |

### Deprecacao em Runtime

```
1. Atualizar description: "DEPRECATED: Use X instead. Sera removido em vY.0.0"
2. Manter tool funcional durante periodo de transicao
3. Documentar migration path no CHANGELOG
4. Remover tool + emitir notifications/tools/list_changed
5. Clients reconectam automaticamente sem restart
```

---

## 7. Evaluation Framework (10 Questoes)

Apos deploy, validar o MCP com 10 questoes que simulam uso real por agents:

1. **Discovery:** "Quais tools estao disponiveis?" — valida tools/list
2. **Simple read:** "Busque os ultimos 5 registros" — valida tool read-only
3. **Filtered read:** "Busque vendas de maio com status APPROVED" — valida filtros
4. **Create:** "Crie um registro com dados X" — valida tool de escrita + idempotency
5. **Update:** "Atualize o registro Y com campo Z" — valida update parcial
6. **Error handling:** "Busque com parametro invalido" — valida Zod error message
7. **Rate limit:** "Execute 10 chamadas rapidas" — valida rate limiting
8. **Pagination:** "Liste todos os registros (mais de 100)" — valida paginacao
9. **Auth error:** "Execute sem token" — valida auth rejection
10. **Complex workflow:** "Encontre o cliente X, liste suas compras e calcule total" — valida multi-tool workflow

Cada questao: PASS (tool chamada corretamente + response util) ou FAIL (com motivo).
Target: 9/10 PASS no primeiro teste.

---

## 8. Graceful Shutdown

```
1. Receber SIGTERM
2. Flaggar server como "shutting down" (retornar 503 para novas requests)
3. Aguardar deadline maximo (30s) para tool calls em andamento
4. Persistir estado de sessao (se stateful)
5. Force-close transports remanescentes
6. process.exit(0)
```

---

## 9. Connection Management (Producao)

| Aspecto | Recomendacao |
|---------|-------------|
| Transport producao | Streamable HTTP (padrao 2026, SSE e legacy) |
| Stateless design | Empurrar estado para Redis/DB externo |
| Load balancer | Stateless funciona com qualquer LB standard |
| Session affinity | Nao necessario se stateless |
| Scaling | Horizontal sem restricoes |
