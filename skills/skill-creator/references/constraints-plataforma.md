# Constraints da Plataforma — Skill Creator

Referência profunda para a Seção 1 do SKILL.md. Todos os limites que afetam design de skills.

---

## 1. Tamanho e Performance (SkillsBench)

### Findings do SkillsBench (arXiv:2602.12670)
- **86 tasks**, 11 domínios, 7 configurações de modelo, 7.308 trajetórias
- **Curated skills:** +16.2pp de melhoria média
- **Melhor uplift:** Claude Code Opus 4.5 — +23.3pp (22% > 45.3%)
- **Pior domínio:** Software Engineering (+4.5pp)
- **Melhor domínio:** Healthcare (+51.9pp)

### Complexidade vs Performance (Tabela 6 do paper)
| Complexidade | Pass Rate | Delta | N |
|-------------|-----------|-------|---|
| Detailed | 42.7% | +18.8pp | 1165 |
| Compact | 37.6% | +17.1pp | 845 |
| Standard | 37.1% | +10.1pp | 773 |
| Comprehensive | 39.9% | -2.9pp | 140 |

**Conclusão:** Compact e Detailed superam Comprehensive por larga margem (~4x). Skills longas e abrangentes PIORAM performance. Skills focadas e concisas MELHORAM.

### Número de Skills por Task
| Skills | Delta | Observação |
|--------|-------|-----------|
| 1 | +10pp | Bom |
| 2-3 | +17-20pp | Ideal |
| 4+ | +5.2pp | Retornos decrescentes |

### Self-generated vs Curated
- Self-generated: melhoria marginal/insignificante
- Curated: melhoria significativa
- **"Effective Skills require human-curated domain expertise that models cannot reliably self-generate."**

### Model Scale Substitution
- Haiku + skills (27.7%) > Opus sem skills (22.0%)
- Implicação: skills bem escritas compensam modelo mais fraco

---

## 2. Progressive Disclosure (Anthropic Official)

### 3 Níveis de Loading
| Nível | Quando Carrega | Custo | Conteúdo |
|-------|---------------|-------|----------|
| L1: Metadata | Sempre (startup) | ~100 tokens/skill | name + description (frontmatter) |
| L2: Instructions | Quando triggered | <5K tokens | SKILL.md body |
| L3: Resources | Sob demanda | Ilimitado | scripts + references/ |

### Token Economics
- 50 skills instaladas = ~5.000 tokens de discovery overhead (L1 only)
- Skill idle = custo zero após L1
- SKILL.md body carregado apenas quando ativado

### Regras Práticas
- SKILL.md < 280 linhas (recomendação Triforce, derivada de <500 linhas Anthropic)
- references/ com no máximo 1 nível de profundidade
- TOC obrigatório em references/ > 100 linhas
- Usar "Consultar references/..." para delegar detalhes ao L3

---

## 3. Frontmatter YAML — Spec Completa (14 campos)

### Campos Padrão (agentskills.io spec)
```yaml
---
name: "skill-name"                    # Identificador único, lowercase, hifenizado
description: >                         # Multi-linha, 2-4 frases
  Descrição completa com triggers de ativação.
  Incluir: nome do agente, cargo, empresa, quando acionar.
allowed-tools:                         # Ferramentas permitidas (opcional)
  - Read
  - Write
  - Edit
---
```

### Campos de Lifecycle (customização Triforce Auto)
```yaml
---
version: "1.0.0"                       # SemVer — MAJOR.MINOR.PATCH
stability: "stable"                    # experimental | beta | stable | deprecated | archived
owner: "nome-do-agente"               # Responsável pela manutenção
next_review: "2026-11-06"             # Data da próxima revisão obrigatória
sources_version: "fonte-v2.1"         # Versão do material-fonte usado
last_updated: "2026-05-06"            # Data da última atualização
last_audit: "2026-05-06"              # Data da última auditoria de qualidade
created: "2026-05-06"                 # Data de criação
deprecated_date: null                  # Preenchido quando stability = deprecated
sunset_date: null                      # Preenchido quando stability = archived
requires: []                           # Dependências de outras skills ["dev-web >= 2.0.0"]
review_reason: "..."                   # Motivo da próxima revisão
changelog: "references/CHANGELOG.md"   # Path pro changelog
---
```

### Regras de Preenchimento
- `next_review` = data de criação/atualização + 180 dias
- `sources_version` deve rastrear a versão EXATA do material usado
- `owner` = id do agente responsável (ex: "dev-web", "copywriter")
- `stability` transitions válidas: experimental > beta > stable > deprecated > archived
- `deprecated_date` + 90 dias = prazo máximo para `sunset_date`

---

## 4. Estrutura de Diretórios

### Padrão Obrigatório
```
.claude/skills/{skill-name}/
  SKILL.md                    # Instrução principal (<280 linhas)
  references/                 # Conhecimento profundo (sem subpastas)
    constraints-plataforma.md
    operacional-ferramentas.md
    estrategico-frameworks.md
    ...
  evals/                      # Suite de avaliação (opcional)
    evals.json
  CHANGELOG.md                # Histórico de mudanças (opcional, pode ficar em references/)
```

### Anti-patterns
- references/sub/sub/ — NÃO. Máximo 1 nível
- SKILL.md > 500 linhas — NÃO. Mover conteúdo para references/
- Scripts executáveis sem auditoria — NÃO. 2.12x mais vulneráveis
- Frontmatter incompleto — NÃO. 14 campos obrigatórios

---

## 5. Segurança

### Dados do Survey Acadêmico (arXiv:2602.12430)
- **42.447 community skills analisadas**
- **26.1% contêm vulnerabilidades**
- **341 skills maliciosas** encontradas em repos comunitários até Feb 2026
- Scripts executáveis: **2.12x mais prováveis** de ter vulnerabilidades

### Four-Tier Trust Model
| Tier | Fonte | Nível de Audit |
|------|-------|---------------|
| T1 | Anthropic oficial | Mínimo (trusted) |
| T2 | Repos curados (daymade, VoltAgent) | Leitura de código |
| T3 | Community skills | Audit completo obrigatório |
| T4 | Skills desconhecidas | Rejeitar por padrão |

### Checklist de Segurança (pré-instalação)
- [ ] Skill NÃO executa scripts sem sandboxing?
- [ ] allowed-tools está minimamente scopado (sem wildcards)?
- [ ] Nenhuma credencial hardcoded no SKILL.md ou references/?
- [ ] Skill não pode escalar privilégios além do necessário?
- [ ] Owner está atribuído e ativo no time?
- [ ] Fonte está no T1 ou T2? Se T3+, audit completo feito?

### OWASP Agentic Skills Top 10 — Itens Críticos
- **AST01/AST02 (Critical):** Full checklist + dynamic testing + manual review
- **AST03-AST05 (High):** Schema validation + static analysis
- **AST09 (Low):** Governance inventory — toda skill deve ter owner
