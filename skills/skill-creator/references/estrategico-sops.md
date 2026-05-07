# SOPs — Skill Creator

Procedimentos operacionais padrão para todas as atividades do Criador de Skills.

---

## SOP 1: Criar Skill do Zero para Novo Funcionário (Pós-Contratação)

### Trigger
Gabriela contrata novo funcionário. Thiago recebe pedido de skill.

### Timeline
- **D+0:** Recebe pedido com: nome, cargo, equipe, reports_to, descrição do gap
- **D+0-1:** Research (8 canais) + gap analysis
- **D+1-2:** Design + Write (SKILL.md + references/)
- **D+2-3:** Eval (baseline + with skill)
- **D+3:** Review (André se técnica, fundador se negócio)
- **D+3-4:** Handoff doc + pilot setup
- **D+4-10:** Pilot com 1 agente (7 dias)
- **D+10:** Rollout ou iterate

### Procedimento

**1. Intake**
```
Ler brief da Gabriela:
  - Nome e cargo do novo funcionário
  - Equipe e supervisor
  - Gaps que a skill deve cobrir
  - Fontes de conhecimento indicadas (cursos, docs, specs)
```

**2. Research (8 canais)**
```
Para cada canal do 8-Channel Search Protocol:
  1. agentskills.io — buscar por keyword do cargo
  2. anthropics/skills — skills oficiais relevantes
  3. daymade/claude-code-skills — skills production-hardened
  4. VoltAgent/awesome-agent-skills — índice geral
  5. openclaw — marketplace alternativo
  6. mcpmarket.com — skills MCP
  7. Perplexity — frameworks e metodologias
  8. Skills internas — .claude/skills/ (reutilização)

Resultado: lista de skills encontradas com avaliação Adopt/Extend/Build
```

**3. Gap Analysis**
```
Ler skills internas dos colegas da mesma equipe:
  - Extrair constraints técnicos comuns (método SCOPE)
  - Identificar interfaces de colaboração
  - Mapear o que o novo cargo NÃO deve fazer (boundaries)
```

**4. Design**
```
Definir:
  - 6 seções e conteúdo de cada uma
  - Frontmatter completo (14 campos)
  - Quais references/ serão necessários
  - Ênfase inviolável (1-2 regras máximas)
  - Progressive disclosure: o que fica no SKILL.md vs references/
```

**5. Write**
```
Criar:
  - SKILL.md (<280 linhas, 6 seções)
  - references/ (detalhes profundos, sem subpastas)
  - evals/evals.json (mínimo 3 cenários com baseline)
```

**6. Review + Handoff**
```
  - PR pro André (skill técnica) ou fundador (skill de negócio)
  - Handoff doc pro agente consumidor
  - Setup de pilot: 1 agente testa por 7 dias
```

---

## SOP 2: Atualizar Skill Existente

### Trigger
- next_review vencido
- sources_version desatualizado (nova versão de framework/SDK)
- Feedback do agente consumidor
- Pedido do André ou fundador

### Classificação: Breaking vs Non-Breaking

| Tipo | Exemplos | Bump |
|------|---------|------|
| **Breaking** | Seção removida, fluxo mudou, constraint invertido | MAJOR |
| **Non-breaking additive** | Nova seção, novo framework, nova ferramenta | MINOR |
| **Non-breaking fix** | Correção de erro, atualização de valor, typo | PATCH |

### Procedimento

**1. Análise de Impacto**
```
Ler SKILL.md atual e identificar:
  - O que mudou na fonte (sources_version)
  - Quais seções são afetadas
  - Quais agentes consumidores serão impactados
  - É breaking change? Se sim, notificar ANTES de atualizar
```

**2. Atualizar**
```
  - Editar SKILL.md (manter <280 linhas)
  - Atualizar references/ afetados
  - Bump version (SemVer)
  - Atualizar frontmatter: last_updated, sources_version, next_review
```

**3. Re-eval (se MAJOR ou MINOR)**
```
  - Re-rodar evals existentes
  - Adicionar novos cenários se conteúdo novo
  - Verificar que delta continua positivo
```

**4. Comunicar**
```
  - Se breaking: handoff doc atualizado + notificar consumidores
  - Se non-breaking: changelog + notificação simples
```

---

## SOP 3: Converter Curso/Material Externo em Skill

### Trigger
Fundador ou Gabriela indica curso/material externo para transformar em skill.

### Procedimento

**1. Ingestão**
```
Ler material completo:
  - Identificar conceitos-chave (10-20 max)
  - Separar: "o que Claude já sabe" vs "o que é novo"
  - Focar APENAS no que é novo ou contra-intuitivo
```

**2. Filtragem (regra Anthropic)**
```
"Only add context Claude doesn't already have."

Descartar:
  - Conceitos básicos que qualquer LLM sabe
  - Explicações pedagógicas (o "por quê" didático)
  - Exemplos genéricos sem contexto específico

Manter:
  - Números específicos (rate limits, tamanhos, timeouts)
  - Breaking changes e gotchas
  - Padrões obrigatórios com código
  - Decisões de arquitetura e trade-offs
  - Anti-patterns documentados
```

**3. Estruturação**
```
Mapear conteúdo filtrado para as 6 seções:
  - Constraints da Plataforma: limites e breaking changes
  - Domínio Operacional: ferramentas e processos
  - Domínio Estratégico: decisões e frameworks
  - Fluxo de Trabalho: ordem de execução
  - Colaboração: interfaces com outros agentes
  - Checklist: itens verificáveis
```

**4. Compressão**
```
Aplicar progressive disclosure:
  - SKILL.md: regras de decisão e resumos (<280 linhas)
  - references/: código completo, exemplos extensos, tabelas detalhadas
```

**5. Eval + Entrega**
```
Seguir steps 5-6 da SOP 1 (eval + review + handoff)
```

---

## SOP 4: Auditoria Periódica do Acervo (Mensal)

### Trigger
Primeira semana de cada mês (automático via next_review tracking).

### Procedimento

**1. Inventário**
```
Glob: .claude/skills/*/SKILL.md
Para cada skill, extrair do frontmatter:
  - name, version, owner, stability, next_review, last_audit, sources_version
Gerar tabela de inventário
```

**2. Check de Vencimento**
```
Para cada skill:
  - next_review < hoje? → ALERTA: revisão vencida
  - last_audit > 90 dias? → ALERTA: audit atrasado
  - owner existe no employees.json? → Se não, ALERTA: skill órfã
```

**3. Redundancy Scan**
```
Para cada par de skills:
  - Comparar descriptions e triggers
  - Se overlap > 60%: ALERTA de overlap
  - Se overlap > 80%: ALERTA CRÍTICO de merge necessário
```

**4. Health Score**
```
Para cada skill com stability = "stable":
  - Calcular health score (6 métricas)
  - Se < 60: ALERTA de qualidade
  - Se < 75: marcado para melhoria
```

**5. Relatório**
```markdown
## Auditoria Mensal de Skills — [mês/ano]

### Resumo
- Total de skills: X
- Production Ready (>=85): X
- Limited Release (75-84): X
- Beta Only (60-74): X
- Reject/Archive (<60): X

### Alertas
- next_review vencidos: [lista]
- Skills órfãs (sem owner): [lista]
- Overlaps críticos: [pares]
- Health score < 60: [lista]

### Ações Recomendadas
1. [ação para cada alerta]
```

---

## SOP 5: Deprecar e Arquivar Skill

### Trigger
- Skill substituída por versão melhor
- Cargo eliminado
- Framework/ferramenta descontinuado

### Timeline de Deprecação
```
D+0:   stability: "deprecated", deprecated_date: "hoje"
D+30:  Primeiro aviso ao time (30 dias restantes)
D+60:  Segundo aviso (30 dias restantes)
D+90:  stability: "archived", sunset_date: "hoje"
       Mover para _archived/ (preservar histórico)
```

### Procedimento

**1. Anúncio**
```
Notificar todos os consumidores da skill:
  - O que está sendo deprecado
  - Por que (substituída? obsoleta?)
  - Alternativa recomendada
  - Timeline (90 dias)
```

**2. Marcar no Frontmatter**
```yaml
stability: "deprecated"
deprecated_date: "2026-05-06"
sunset_date: "2026-08-04"  # +90 dias
```

**3. Arquivar**
```
No sunset_date:
  - Mover skill para .claude/skills/_archived/{skill-name}/
  - Atualizar employees.json se o cargo foi eliminado
  - Registrar no changelog
```

---

## SOP 6: Handoff de Skill Nova pro Agente Consumidor

### Trigger
Skill passou no review e está pronta para uso.

### 5 Fases do Handoff

```
1. DRAFT      → Skill-creator produziu SKILL.md + references/ + evals
2. REVIEW     → André (técnica) ou fundador (negócio) aprovou
3. PILOT      → 1 agente testa por 7 dias, reporta issues
4. ROLLOUT    → Skill disponibilizada para todo o time relevante
5. MONITOR    → Feedback loop contínuo (30/60/90 dias)
```

### Procedimento

**1. Criar Handoff Doc** (template na SOP)

**2. Selecionar Agente Piloto**
```
Critérios:
  - É o principal consumidor da skill
  - Está ativo (status: hired, onboarded_at preenchido)
  - Tem tasks pendentes onde a skill seria útil
```

**3. Pilot (7 dias)**
```
  - Agente usa a skill em produção
  - Registrar: sucessos, falhas, confusões, sugestões
  - Thiago acompanha e ajusta se necessário
```

**4. Rollout**
```
  - Comunicar ao time: "skill X disponível para uso"
  - Atualizar employees.json com a skill nas skills[] dos consumidores
  - Changelog atualizado
```

**5. Monitor (30/60/90)**
```
  - 30 dias: quick check — skill sendo usada? Issues?
  - 60 dias: feedback formal do consumidor principal
  - 90 dias: primeiro audit completo (health score)
```

---

## SOP 7: Consultar Skills Técnicas Internas (Método SCOPE)

### Trigger
Precisa extrair constraints técnicos de skills que já existem para incluir em skill nova.

### Framework SCOPE (IdeaPlan, adaptado)
- **S**tructure: Qual a arquitetura documentada?
- **C**onstraints: Quais os limites Always/Ask First/Never?
- **O**utput: Qual o formato de entrega esperado?
- **P**atterns: Quais padrões obrigatórios? Quais anti-patterns?
- **E**xamples: Quais exemplos concretos estão na skill?

### Procedimento (3 passos)

**Passo 1: Leitura Automatizada**
```
Ler skills internas relevantes e extrair constraints em 3 categorias:

ALWAYS (regras que sempre se aplicam):
  - "Sempre use Next.js App Router" (extraído de dev-lider)
  - "Sempre prepare: false com pooler" (extraído de dev-backend)

ASK FIRST (decisões que precisam aprovação):
  - "Proponha novas deps antes de instalar" (dev-lider)
  - "Schema changes precisam do André" (dev-backend)

NEVER (proibições absolutas):
  - "Nunca raw SQL, use Drizzle ORM" (dev-lider)
  - "Nunca .enableRLS(), usar .withRLS()" (dev-backend)
```

**Passo 2: Cruzar com Skills dos Colegas**
```
Skills a consultar por equipe:

equipe-sistemas:
  - .claude/skills/dev-lider/SKILL.md
  - .claude/skills/dev-backend/SKILL.md
  - .claude/skills/dev-frontend/SKILL.md
  - .claude/skills/dev-web/SKILL.md
  - .claude/skills/luna-qa/SKILL.md
  - .claude/skills/revisor-sistemas/SKILL.md
  - .claude/skills/mcp-creator/SKILL.md

marketing:
  - .claude/skills/copywriter/SKILL.md
  - .claude/skills/curador-ia/SKILL.md
  - .claude/skills/designer-instagram/SKILL.md
  - .claude/skills/social-media/SKILL.md

comercial:
  - .claude/skills/estrategista-comercial/SKILL.md
  - .claude/skills/prospector/SKILL.md
```

**Passo 3: Validação Cruzada**
```
Comparar constraints extraídos (passo 1) com constraints das skills dos colegas (passo 2):
  - Discrepância = gap de documentação
  - Feedback: atualizar skill do colega se constraint faltando
  - Resultado: lista consolidada de constraints para a skill nova
```
