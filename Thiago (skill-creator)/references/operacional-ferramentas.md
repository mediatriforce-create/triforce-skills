# Operacional e Ferramentas — Skill Creator

Referência profunda para a Seção 2 do SKILL.md. Anatomia, templates, ferramentas e processos.

---

## 1. Anatomia de SKILL.md (6 Seções Obrigatórias)

### Seção 1: Constraints da Plataforma
**Propósito:** Limites técnicos que o agente PRECISA respeitar.
- Tabelas com limites numéricos (rate limits, timeouts, tamanhos)
- Breaking changes de dependências
- Gotchas que causam bugs silenciosos

**Exemplo:**
```markdown
### Vercel (Hobby)
| Limite | Valor |
|--------|-------|
| Serverless max duration | 60s |
| Request/response body | 4.5 MB |
```

### Seção 2: Domínio Operacional
**Propósito:** O que o agente tem disponível e o que produz.
- MCPs ativos (com namespaces)
- Skills locais e externas
- Inputs e outputs do cargo
- Ferramentas e quando usar cada uma

### Seção 3: Domínio Estratégico
**Propósito:** Como o agente DECIDE entre opções.
- Regras de decisão (if X then Y)
- Padrões obrigatórios com código mínimo
- Frameworks de decisão
- Anti-patterns explícitos

### Seção 4: Fluxo de Trabalho
**Propósito:** Em que ORDEM executar.
- STEP 0 obrigatório (ler ops/)
- Fluxos numerados com steps sequenciais
- Tom baseado em seniority
- STEP FINAL (atualizar ops/)

### Seção 5: Colaboração com o Time
**Propósito:** Com quem o agente interage e como.
- Tabela: Colega | Role | Como interage
- Quem recebe entregas
- Quem faz review
- Quem escalar em caso de dúvida

### Seção 6: Checklist de Entrega
**Propósito:** Itens verificáveis antes de entregar qualquer trabalho.
- Ênfase inviolável com itens dedicados
- Qualidade estrutural
- Segurança e manutenção
- 10-14 itens no total

---

## 2. Frontmatter Completo (14 Campos)

```yaml
---
# === Identidade ===
name: "skill-name"                     # lowercase, hifenizado, único
description: >                          # 2-4 frases com triggers de ativação
  [Nome], [Cargo] da [equipe] da Triforce Auto.
  [O que faz em 1 frase].
  Acionar para: [lista de triggers separados por vírgula].

# === Versionamento ===
version: "1.0.0"                        # SemVer: MAJOR.MINOR.PATCH
last_updated: "2026-05-06"             # ISO 8601
created: "2026-05-06"                  # ISO 8601

# === Rastreabilidade ===
sources_version: "Framework X v2.1 | Tool Y 2026"  # Versões exatas das fontes
next_review: "2026-11-06"              # +180 dias da criação/última atualização
review_reason: "Motivo específico"      # O que pode mudar até a próxima revisão
owner: "skill-id-do-responsável"        # Quem mantém esta skill

# === Lifecycle ===
stability: "stable"                     # experimental | beta | stable | deprecated | archived
last_audit: "2026-05-06"              # Última auditoria de qualidade
deprecated_date: null                   # Quando entrou em deprecated
sunset_date: null                       # Quando será/foi arquivada

# === Dependências ===
requires: []                            # ["dev-web >= 2.0.0"]
changelog: "references/CHANGELOG.md"    # Path pro changelog
---
```

### Regras de SemVer para Skills
- **MAJOR (X.0.0):** Reestruturação completa, seções removidas, breaking changes de processo
- **MINOR (1.X.0):** Nova seção, novo framework adicionado, nova ferramenta
- **PATCH (1.0.X):** Correção de erros, atualização de valores, ajuste de redação

---

## 3. Adopt/Extend/Build Decision Matrix

### 8-Channel Search Protocol (antes de criar do zero)

| Canal | Onde Buscar | O que procurar |
|-------|-----------|---------------|
| 1 | agentskills.io | Skills no marketplace oficial |
| 2 | github.com/anthropics/skills | Skills oficiais Anthropic |
| 3 | github.com/daymade/claude-code-skills | Skills production-hardened |
| 4 | github.com/VoltAgent/awesome-agent-skills | Índice de 549+ skills |
| 5 | openclaw.nasseroumer.com | Marketplace alternativo |
| 6 | mcpmarket.com/tools/skills | Skills do MCP market |
| 7 | Perplexity search | Skills recentes não indexadas |
| 8 | Skills internas (.claude/skills/) | Reutilização dentro da empresa |

### Decisão Adopt/Extend/Build

| Critério | Adopt | Extend | Build |
|----------|-------|--------|-------|
| Cobertura do gap | >80% | 50-80% | <50% |
| Qualidade (audit score) | >65/80 | >50/80 | N/A |
| Manutenção ativa | Sim | Sim | N/A |
| Customização necessária | Mínima | Moderada | Total |
| Ação | Instalar e usar | Fork, customizar, manter | Criar do zero |

---

## 4. 9 Checkpoint Questions (daymade)

Antes de finalizar qualquer skill nova, responder:

1. **O nome é claro e sem ambiguidade?** (lowercase, hifenizado, descreve o que faz)
2. **A description cobre todos os triggers de ativação?** (quando o agente deve ser acionado)
3. **O frontmatter está completo?** (14 campos preenchidos)
4. **O SKILL.md está dentro do limite?** (<280 linhas, progressive disclosure aplicado)
5. **As 6 seções estão presentes?** (Constraints, Operacional, Estratégico, Fluxo, Colaboração, Checklist)
6. **O path integrity está correto?** (references/ existem, links não quebrados)
7. **O conteúdo é conciso e acionável?** (sem repetição, sem fluff, degrees of freedom corretos)
8. **A segurança foi verificada?** (sem credenciais, sem PII, allowed-tools scopado)
9. **O eval baseline foi medido?** (antes vs depois, mínimo 3 cenários)

---

## 5. Template de Eval (evals.json)

```json
{
  "skill": "skill-name",
  "version": "1.0.0",
  "date": "2026-05-06",
  "evaluator": "skill-creator",
  "scenarios": [
    {
      "id": "eval-001",
      "description": "Cenário representativo do uso principal",
      "input": "Prompt ou task que o agente recebe",
      "expected_behavior": "O que o agente deve fazer/produzir",
      "baseline_without_skill": {
        "passed": false,
        "notes": "Falha específica observada sem a skill"
      },
      "result_with_skill": {
        "passed": true,
        "notes": "Comportamento correto observado com a skill"
      }
    }
  ],
  "summary": {
    "total_scenarios": 3,
    "baseline_pass_rate": "33%",
    "skill_pass_rate": "100%",
    "delta": "+67pp"
  }
}
```

### Critérios de Eval Mínimos
- Mínimo 3 cenários por skill
- Pelo menos 1 cenário edge case
- Baseline SEMPRE medido antes de escrever a skill
- Delta positivo obrigatório (skill deve melhorar, não piorar)

---

## 6. Template de Handoff Document

```markdown
## Handoff: [nome-da-skill]

### Quick Summary
- O que: [descrição em 1 frase]
- Para quem: [agentes que vão usar]
- Por que agora: [gap que resolve]

### Status
- Fase: [DRAFT | REVIEW | PILOT | ROLLOUT]
- Progresso: [%]
- Blockers: [lista ou "nenhum"]
- Próxima ação: [descrição]

### Contexto
- Gap original: [referência ao gap analysis]
- Fontes usadas: [lista de fontes com versões]
- Decisões tomadas: [escolhas e racional]

### Instruções de Uso
- Trigger: [como ativar a skill — palavras-chave, contextos]
- Exemplo: [caso de uso concreto com input/output esperado]
- Limitações: [o que a skill NÃO faz]

### Feedback Solicitado (7 dias de pilot)
- [ ] Instruções claras?
- [ ] Exemplos suficientes?
- [ ] Constraints precisos?
- [ ] Falta algum caso de uso?
- [ ] Algum comportamento inesperado?
```

---

## 7. Marketplace — Onde Buscar Skills

| Fonte | URL | Tipo |
|-------|-----|------|
| Anthropic oficial | github.com/anthropics/skills | T1 (trusted) |
| agentskills.io | agentskills.io | Spec + marketplace |
| daymade | github.com/daymade/claude-code-skills | T2 (curated, 900+ stars) |
| VoltAgent | github.com/VoltAgent/awesome-agent-skills | Índice (549+ skills) |
| OpenClaw | openclaw.nasseroumer.com | Marketplace alternativo |
| MCP Market | mcpmarket.com/tools/skills | Skills do MCP ecosystem |
| Composio | github.com/ComposioHQ/awesome-claude-skills | Lista curada |
| ScienceAIX | github.com/scienceaix/agentskills | Referência acadêmica |

---

## 8. Security Gate — Checklist de Audit para Skills Externas

### Pré-instalação (obrigatório para T3+)

**Pass 1: Structural Scan**
- [ ] SKILL.md tem frontmatter válido?
- [ ] Tamanho dentro dos limites (<500 linhas)?
- [ ] references/ sem profundidade excessiva?

**Pass 2: Content Review**
- [ ] Nenhum script executa código não sanitizado?
- [ ] Nenhuma credencial ou token hardcoded?
- [ ] allowed-tools está minimamente scopado?
- [ ] Nenhum acesso a rede sem justificativa?

**Pass 3: Behavioral Assessment**
- [ ] A skill faz o que diz fazer? (testar 1-2 cenários)
- [ ] Outputs são determinísticos e esperados?
- [ ] Nenhum side effect inesperado?

**Resultado:**
- 3 passes OK = instalar
- 1+ falha = rejeitar ou remediar antes de instalar
