# Frameworks Estratégicos — Skill Creator

Referência profunda para a Seção 3 do SKILL.md. Todos os frameworks de decisão.

---

## 1. Framework 10 Componentes de Prompt Engineering

Fonte: claude-code-handbook (ThamJiaHe), adaptado para skills.

| # | Componente | O que define | Exemplo em SKILL.md |
|---|-----------|-------------|---------------------|
| 1 | Role definition | Quem o agente É | "Você é Gabriel, Backend Developer Sênior..." |
| 2 | Context setting | Ambiente e constraints | Seção 1 (Constraints da Plataforma) |
| 3 | Task specification | O que FAZER | Seção 2 (Domínio Operacional — outputs) |
| 4 | Output formatting | Como ENTREGAR | Templates, code blocks, tabelas |
| 5 | Constraint definition | O que NÃO fazer | Anti-patterns, limites, "nunca..." |
| 6 | Quality metrics | Como MEDIR sucesso | Checklist de entrega (Seção 6) |
| 7 | Thinking parameters | Quando pensar profundo | Decisões de arquitetura, edge cases |
| 8 | Few-shot examples | Exemplos concretos | Code snippets no Domínio Estratégico |
| 9 | XML structuring | Organização hierárquica | Seções, subseções, tabelas Markdown |
| 10 | Agentic patterns | Workflow de múltiplos steps | Seção 4 (Fluxo de Trabalho) |

### Aplicação Prática
Ao criar uma skill, verificar que todos os 10 componentes estão representados:
- Componentes 1-3: frontmatter + Seção 1-2
- Componentes 4-6: Seção 2 (outputs) + Seção 6 (checklist)
- Componentes 7-8: Seção 3 (decisões + exemplos)
- Componentes 9-10: Estrutura geral + Seção 4

---

## 2. Degrees of Freedom (Anthropic Official)

### Definição
O grau de liberdade determina quão prescritiva a instrução deve ser.

| Grau | Quando usar | Formato da instrução | Exemplo |
|------|-------------|---------------------|---------|
| **High** | Decisões heurísticas, múltiplas abordagens válidas | Princípios e guidelines | "Priorize clareza sobre criatividade" |
| **Medium** | Padrão preferido com variação aceitável | Pseudocode ou template | "Use este template, adapte ao contexto" |
| **Low** | Operações frágeis, consistência crítica | Script exato, passo-a-passo | "Execute exatamente: step 1, step 2, step 3" |

### Analogia
- **Low freedom:** Ponte estreita — um caminho, sem desvios
- **High freedom:** Campo aberto — múltiplos caminhos, mesmo destino

### Regras de Aplicação em SKILL.md
- **Seção 1 (Constraints):** Low freedom — limites são absolutos
- **Seção 3 (Estratégico):** Medium freedom — frameworks com flexibilidade
- **Seção 4 (Fluxo):** Medium-Low — ordem importa, mas detalhes variam
- **Seção 5 (Colaboração):** High freedom — contexto social é dinâmico

---

## 3. Claude A/B Pattern (Anthropic Official)

### Conceito
Usar duas instâncias de Claude com papéis diferentes para refinar skills.

```
Claude A (design time)          Claude B (runtime, fresh)
  |                                |
  | Escreve skill v0.1             |
  |------------------------------->|
  |                                | Testa a skill em cenários
  |                                | Reporta falhas
  |<-------------------------------|
  | Analisa falhas                 |
  | Refina skill v0.2              |
  |------------------------------->|
  |                                | Re-testa
  |         (ciclo até convergir)  |
```

### Regras
- Claude A tem CONTEXTO COMPLETO (fontes, pesquisa, histórico)
- Claude B é FRESH — sem contexto prévio, só a skill
- Se Claude B falha, o problema está na skill, não no Claude
- Mínimo 2 ciclos A/B antes de declarar skill pronta
- Claude B deve ser testado com cenários que o criador NÃO escreveu

---

## 4. Eval-Driven Development

### Pipeline Completo

```
1. IDENTIFY GAP
   Agente falha em task sem skill
   Documentar: qual task, qual falha, qual output esperado

2. CREATE EVAL SUITE
   Mínimo 3 cenários:
   - Happy path (caso padrão)
   - Edge case (situação incomum)
   - Adversarial (tentativa de quebrar)

3. MEASURE BASELINE
   Rodar Claude sem skill nos 3+ cenários
   Registrar: pass/fail, qualidade do output, tempo

4. WRITE MINIMAL SKILL
   Escrever APENAS o necessário para cobrir os gaps
   "Only add context Claude doesn't already have"

5. MEASURE WITH SKILL
   Rodar Claude COM skill nos mesmos cenários
   Registrar: pass/fail, qualidade, tempo

6. COMPARE
   Delta deve ser positivo
   Se negativo: skill está atrapalhando, simplificar

7. ITERATE
   Refinar skill baseado nos resultados
   Repetir steps 5-6 até convergir
```

### Anti-patterns de Eval
- Criar eval DEPOIS de escrever a skill (viés de confirmação)
- Testar apenas happy path (falsa confiança)
- Não medir baseline (sem referência de comparação)
- Adicionar conteúdo sem re-avaliar (skill bloat)

---

## 5. Lifecycle 8 Stages

### Stages com Critérios de Transição

```
1. IDENTIFY    ──> Existe gap documentado no time?
                   SIM → próximo stage
                   NÃO → não criar skill

2. RESEARCH    ──> Busca nos 8 canais completa?
                   SIM → decisão Adopt/Extend/Build
                   NÃO → continuar buscando

3. BUILD       ──> SKILL.md + references/ + evals criados?
                   SIM → próximo stage
                   NÃO → continuar escrevendo

4. PILOT       ──> 1 agente testou por 7 dias?
                   SIM, sem issues → próximo stage
                   SIM, com issues → voltar para BUILD
                   NÃO → aguardar

5. DEPLOY      ──> Rollout para time completo feito?
                   SIM → próximo stage
                   NÃO → comunicar e distribuir

6. MONITOR     ──> Usage tracking ativo? next_review agendado?
                   SIM → manter
                   ALERTA → voltar para BUILD (atualização)

7. DEPRECATE   ──> Skill substituída ou obsoleta?
                   SIM → sunset timeline (30-60-90 dias)
                   NÃO → manter em MONITOR

8. ARCHIVE     ──> Sunset date atingido?
                   SIM → mover para _archived/
                   NÃO → manter aviso de deprecated
```

### Campos de Frontmatter por Stage
| Stage | stability | deprecated_date | sunset_date |
|-------|-----------|----------------|-------------|
| 1-3 | experimental | null | null |
| 4 | beta | null | null |
| 5-6 | stable | null | null |
| 7 | deprecated | "2026-XX-XX" | "2026-XX-XX" (+90d) |
| 8 | archived | "2026-XX-XX" | "2026-XX-XX" |

---

## 6. Health Score — 6 Métricas Ponderadas

### Fórmula
```
HealthScore = (Uso * 0.25) + (Trigger * 0.20) + (Overlap * 0.20) +
              (Confiabilidade * 0.15) + (Tokens * 0.10) + (Manutenção * 0.10)
```

### Detalhamento por Métrica

| # | Métrica | Peso | 100 (max) | 0 (min) |
|---|---------|------|-----------|---------|
| 1 | Frequência de uso | 25% | Core workflow, usado diariamente | 0 usos em 90 dias |
| 2 | Clareza de trigger | 20% | Triggers explícitos, description específica | Trigger vago, confunde com outras skills |
| 3 | Penalidade de overlap | 20% | 0% overlap com outras skills | 80%+ overlap (candidata a merge) |
| 4 | Confiabilidade | 15% | 100% taxa de sucesso em evals | <50% taxa de sucesso |
| 5 | Eficiência de tokens | 10% | <150 linhas, progressive disclosure | >500 linhas, sem references/ |
| 6 | Qualidade de manutenção | 10% | Owner ativo, next_review em dia, links OK | Sem owner, review vencido, links quebrados |

### Disposições
| Score | Disposição | Ação |
|-------|-----------|------|
| >= 85 | **Production Ready** | Manter, monitorar |
| 75-84 | **Limited Release** | Fix minor issues |
| 60-74 | **Beta Only** | Needs significant work |
| < 60 | **Reject/Archive** | Rewrite ou aposentar |

---

## 7. Veto Gates (adaptado MedSkillAudit)

### Veto Gate 1: Structural Audit (hard gates)
Falha em QUALQUER item = skill rejeitada.

- [ ] SKILL.md existe e tem frontmatter válido (YAML parseia sem erro)
- [ ] Campos obrigatórios presentes: name, description, version, owner, stability, next_review
- [ ] Nenhum script executa código não sanitizado
- [ ] allowed-tools está scopado (sem wildcards)
- [ ] Tamanho do SKILL.md < 500 linhas
- [ ] references/ existem e links não estão quebrados

### Veto Gate 2: Domain Audit (gates de agência digital)
Falha em QUALQUER item = skill rejeitada.

- [ ] Não gera conteúdo que viola brand guidelines de clientes
- [ ] Não expõe PII, API keys, ou dados sensíveis
- [ ] Não faz claims legais/financeiros sem disclaimer
- [ ] Output boundaries claros (skill sabe o que NÃO fazer)

### Scoring Post-Veto
Se passou nos 2 veto gates:
```
FinalScore = (0.4 * S_static) + (0.6 * D_dynamic)
S_static  = Health Score (6 métricas, 0-100)
D_dynamic = Performance média em evals (0-100)
```

---

## 8. Redundancy Detection

### Método: Trigger Collision Matrix

**Step 1:** Extrair triggers de todas as skills (description + acionar para)
**Step 2:** Comparar pares de skills por similaridade de triggers
**Step 3:** Classificar overlap:

| Overlap | Classificação | Ação |
|---------|--------------|------|
| < 30% | Saudável | Nenhuma |
| 30-60% | Complementar | Documentar boundary na Seção 5 |
| 60-80% | Alerta | Review de merge obrigatório |
| > 80% | Crítico | Merge ou eliminar uma |

### Merge Patterns (AIXplore)
- **Mode-based:** Mesmo workflow, outputs diferentes → consolidar com flag de modo
- **Feature-based:** Várias skills são features de uma capacidade → skill única com seções
- **Audience-based:** Só muda formato final → consolidar com template pattern

### Exemplo Prático
Se `dev-web` e `dev-frontend` tivessem 70% de overlap em triggers de CSS/Tailwind:
1. Documentar boundary: dev-web = LPs de clientes, dev-frontend = sistema interno
2. Cada SKILL.md declara explicitamente o que NÃO faz
3. Se boundary insuficiente: considerar merge com mode flag
