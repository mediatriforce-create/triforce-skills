# SOPs — Procedimentos Operacionais Padrao

**Ultima atualizacao:** 2026-05-06

---

## SOP 1: Criar MCP Novo do Zero

### Trigger
Pedido para criar MCP server para API sem spec ou sem MCP existente.

### Passos

1. **Research**
   - Documentacao oficial da API target
   - Rate limits, auth, endpoints, webhooks
   - Buscar no mcpmarket.com, mcp.directory, pulsemcp.com se ja existe
   - Se existe: avaliar encapsular vs construir do zero

2. **Design**
   - Listar tools necessarias (agent-centric, nao API wrappers)
   - Definir Zod schemas para cada tool
   - Definir tool annotations (readOnlyHint, destructiveHint, idempotentHint)
   - Escolher framework: SDK oficial (complexo) ou FastMCP (simples/medio)
   - Escolher transport: stdio (dev) ou Streamable HTTP (producao)
   - Escolher plataforma: Vercel / Cloudflare / Supabase Edge

3. **Scaffold**
   ```bash
   mkdir -p mcp-server-nome/{src/{tools,lib,resources,prompts},transports,tests}
   cd mcp-server-nome
   npm init -y
   npm install @modelcontextprotocol/sdk zod  # ou: npm install fastmcp zod
   npm install -D typescript @types/node
   ```

4. **Implement**
   - `src/server.ts` — instancia McpServer/FastMCP
   - `src/tools/*.ts` — um arquivo por tool ou grupo de tools
   - `src/lib/api-client.ts` — HTTP client com retry + auth
   - `src/lib/schemas.ts` — Zod schemas compartilhados
   - Rate limiter se API target tem limites

5. **Test**
   - MCP Inspector: `npx -y @modelcontextprotocol/inspector`
   - Smoke test sem token (imports, tool count, lifecycle)
   - Live test com token (1-2 chamadas read-only)

6. **Document**
   - CHANGELOG.md (Keep a Changelog)
   - README.md com setup, config, tools disponiveis
   - Schema docs: tabela tools/inputs/outputs

7. **Review**
   - PR para Andre (dev-lider)
   - Rodrigo valida seguranca
   - Luna valida testes
   - CI verde

8. **Deploy**
   - Plataforma escolhida no passo 2
   - Health check ativo
   - OpenTelemetry configurado

9. **Register**
   - Adicionar ao mcp.json do projeto ou ~/.claude.json
   - Comunicar ao time como consumir

### Output
MCP server em producao, documentado, testado e registrado.

---

## SOP 2: Criar MCP a Partir de API Spec

### Trigger
Spec OpenAPI, Postman collection ou SDK codegen disponivel.

### Passos

1. **Detectar formato da spec**
   | Formato | Indicador | Adapter |
   |---------|-----------|---------|
   | OpenAPI/Swagger | `paths:` no YAML/JSON | OpenAPI adapter |
   | Postman Collection | `item[].request` | Postman adapter |
   | SDK Codegen | `apis[]` e `fields[]` | Direto |

2. **Avaliar Speakeasy**
   - Se spec OpenAPI >= 3.0: considerar Speakeasy para geracao automatica
   - `speakeasy generate --lang typescript --schema spec.yaml`
   - Customizar com `x-speakeasy-mcp` extensions
   - Scoping: `--scope read` para tools read-only

3. **Se Speakeasy nao for viavel**
   - Seguir SOP 1 usando a spec como referencia
   - Mapear cada endpoint para uma tool agent-centric
   - Nao criar 1:1 endpoint-tool — agrupar logicamente

4. **Verificar cobertura**
   - Todos os endpoints criticos tem tool?
   - Webhooks documentados como Resources?
   - Rate limits encapsulados?

5. **Continuar com passos 5-9 da SOP 1**

### Output
MCP server gerado (parcial ou total) a partir de spec, com customizacoes agent-centric.

---

## SOP 3: Atualizar MCP Existente

### Trigger
Feature request, bug fix ou atualizacao de API target.

### Classificar: Breaking vs Non-Breaking

| Tipo | Exemplos | Versao |
|------|----------|--------|
| **Non-breaking** | Nova tool, campo opcional adicionado, bug fix | MINOR ou PATCH |
| **Breaking** | Tool removida, campo required adicionado, schema alterado, auth mudou | MAJOR |

### Non-Breaking Change

1. Implementar mudanca
2. Adicionar testes
3. Atualizar CHANGELOG.md (Added/Changed/Fixed)
4. PR com label `non-breaking`
5. Bump MINOR ou PATCH
6. Deploy

### Breaking Change

1. **NAO** modificar tool/schema existente in-place
2. Criar nova tool com novo nome (ex: `query_records_v2`)
3. Marcar tool antiga como DEPRECATED na description
4. Documentar migration path no CHANGELOG
5. PR com label `breaking-change` + descricao detalhada
6. Bump MAJOR
7. Deploy com periodo de transicao (manter ambas)
8. Apos periodo: remover tool antiga + emitir `notifications/tools/list_changed`

---

## SOP 4: Deprecar Tool em MCP Existente

### Trigger
Tool precisa ser substituida ou removida.

### Passos

1. **Atualizar description da tool deprecated**
   ```typescript
   server.registerTool('search_records', {
     description: 'DEPRECATED: Use query_records_v2 instead. This tool will be removed in v3.0.0.',
     // ... manter implementacao funcional
   });
   ```

2. **Documentar no CHANGELOG**
   ```markdown
   ### Tool Contracts
   - **DEPRECATED** `search_records` — sera removido em v3.0.0. Usar `query_records_v2`.
   ```

3. **Comunicar ao time**
   - Avisar Gabriel, Diego, Felipe que usam o MCP
   - Dar prazo para migracao

4. **Apos periodo de transicao**
   ```typescript
   server.removeTool('search_records');
   // SDK emite notifications/tools/list_changed
   // Clients re-chamam tools/list automaticamente
   ```

5. **CHANGELOG**
   ```markdown
   ### Breaking Changes
   - **REMOVED** `search_records` (use `query_records_v2`)
   ```

6. **Bump MAJOR version**

---

## SOP 5: Debug de MCP com Inspector

### Trigger
MCP server nao funciona, tools retornam erro, ou comportamento inesperado.

### Passos

1. **Iniciar Inspector**
   ```bash
   npx -y @modelcontextprotocol/inspector
   ```

2. **Conectar ao server**
   - stdio: `node dist/server.js`
   - HTTP: `http://localhost:3000/api/mcp`

3. **Checklist de diagnostico**

   | Passo | O que verificar | Se falhar |
   |-------|----------------|-----------|
   | 1 | Conexao estabelece? | Verificar transport, porta, command |
   | 2 | `initialize` response OK? | Verificar capabilities, serverInfo |
   | 3 | `tools/list` retorna tools? | Verificar registracao de tools |
   | 4 | Schemas Zod corretos? | Verificar inputSchema |
   | 5 | Tool read-only funciona? | Verificar handler, API client, auth |
   | 6 | Tool de escrita funciona? | Verificar permissions, Zod validation |
   | 7 | Erro retorna mensagem util? | Verificar error formatting |
   | 8 | Rate limiting funciona? | Testar burst de chamadas |

4. **Logs**
   - Server logs: verificar stderr output
   - Inspector: verificar JSON-RPC messages (request/response)
   - API target: verificar status codes (401? 429? 500?)

5. **Problemas comuns**

   | Sintoma | Causa provavel | Solucao |
   |---------|---------------|---------|
   | "Connection refused" | Server nao iniciou | Verificar comando, porta |
   | "Tool not found" | Tool nao registrada | Verificar registerTool/addTool |
   | Schema validation error | Zod schema errado | Verificar z.object() contra API |
   | 401 Unauthorized | Token invalido/expirado | Renovar token, verificar env var |
   | 429 Too Many Requests | Rate limit da API target | Implementar Bottleneck |
   | Timeout | API lenta ou server travou | Verificar duration limits da plataforma |

---

## SOP 6: Deploy Multi-Plataforma

### Trigger
MCP pronto para producao. Escolher plataforma baseado no contexto.

### Decisao de plataforma

| Contexto | Plataforma | Motivo |
|----------|-----------|--------|
| Ja tem app Next.js na Vercel | **Vercel** | @vercel/mcp-adapter, zero-ops |
| Precisa de edge global + stateful | **Cloudflare** | McpAgent + Durable Objects |
| Precisa acessar Supabase DB diretamente | **Supabase Edge** | SUPABASE_DB_URL auto-injetado |
| Dev/teste local | **stdio** | Sem deploy necessario |
| API com latencia critica | **Cloudflare** | Edge global, menor latencia |

### Checklist pre-deploy

- [ ] Health check endpoint ativo
- [ ] Env vars configuradas na plataforma
- [ ] CORS configurado (se HTTP remoto)
- [ ] Auth configurado (OAuth 2.1 ou Bearer token)
- [ ] Rate limiting ativo
- [ ] OpenTelemetry configurado
- [ ] CHANGELOG atualizado
- [ ] CI verde

### Checklist pos-deploy

- [ ] Endpoint acessivel
- [ ] MCP Inspector conecta com sucesso
- [ ] 1 tool read-only funciona
- [ ] Health check retorna "healthy"
- [ ] Logs fluindo no OpenTelemetry / dashboard da plataforma
- [ ] Registrado no mcp.json / claude.json

---

## SOP 7: Onboarding de Novo MCP no Time

### Trigger
MCP server novo deployado e pronto para consumo.

### Passos

1. **Documentar**
   - README com: o que faz, quais tools, como configurar
   - Tabela de tools com inputs/outputs
   - Exemplos de uso (prompts que acionam cada tool)

2. **Registrar**
   - Adicionar ao mcp.json do projeto (se projeto-especifico)
   - Ou adicionar ao ~/.claude.json (se global)
   - Env vars: documentar quais sao necessarias e onde obter

3. **Comunicar**
   - Gabriel (dev-backend): "MCP X disponivel, tools: Y, Z. Para usar: ..."
   - Diego (dev-frontend): "MCP X documenta schemas em: ..."
   - Felipe (dev-web): "MCP X para integracao com servico W: ..."
   - Luna (luna-qa): "Testes do MCP X passam no Inspector. Se quiser E2E: ..."
   - Rodrigo (revisor-sistemas): "PR do MCP X revisado e aprovado. Changelog em: ..."

4. **Verificar consumo**
   - Acompanhar primeiras chamadas via OpenTelemetry
   - Coletar feedback do time nas primeiras 48h
   - Ajustar descriptions de tools se agents nao encontram a tool certa

5. **Atualizar ops/**
   - Se MCP usa conta/servico novo: atualizar accounts.yaml
   - Se MCP e critico: considerar alerta em alerts.yaml
