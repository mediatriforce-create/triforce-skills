# Tech Lead — Práticas para Startup Brasileira (Equipe Pequena)

## Responsabilidades do Cargo

André é **player-coach**: executa código E lidera. Balanço esperado:
- Fase inicial do sistema: 60-70% codando, 30-40% liderando
- Sistema estável: 40-50% codando, 50-60% arquitetura/review/gestão

**Nunca:** delegar responsabilidades sem clareza. **Sempre:** definir quem decide o quê antes de executar.

---

## Code Review

**SLA:** 24h para revisar qualquer PR aberto pela equipe (exceto feriados).

**Checklist de review:**
1. Lógica correta para o caso de uso descrito
2. TypeScript sem `any` não justificado
3. RLS ativo em tabelas com dados sensíveis
4. `prepare: false` em conexões Drizzle com pooler
5. Sem secrets hardcoded
6. Testes para lógica nova (quando aplicável)
7. Documentação atualizada se mudou comportamento

**Feedback:** específico e acionável. Nunca pessoal. Exemplo correto:
> "Essa query pode retornar N+1 quando tem muitos leads. Sugestão: usar `.with()` para join em uma única query — ver exemplo em `references/drizzle-supabase.md`."

---

## Cadência de Sprints (2 semanas)

| Evento | Duração | Frequência |
|--------|---------|-----------|
| Sprint Planning | 1h | Início de cada sprint |
| Daily stand-up | 15min | Diário |
| Sprint Review (demo) | 30min | Fim de cada sprint |
| Retrospectiva | 30min | Fim de cada sprint |
| Backlog Refinement | 45min | 1x por sprint (meio) |

**Buffer obrigatório:** Reservar 15% da capacidade do sprint para bugs críticos e trabalho não planejado. Nunca planejar 100% da capacidade.

---

## Definition of Done (DoD)

Uma tarefa só é "feita" quando:
- [ ] Código compila e CI verde (lint + typecheck + build)
- [ ] PR revisado e aprovado
- [ ] Deploy em staging verificado
- [ ] Fundador ou responsável pela feature confirmou que atende ao requisito
- [ ] Documentação atualizada (ADR se decisão arquitetural)
- [ ] Sem regressões conhecidas

---

## ADR — Architecture Decision Records

**Quando criar:** toda decisão com impacto duradouro na arquitetura.

**Template:**
```markdown
# ADR-{N}: {Título}
Data: {YYYY-MM-DD}
Status: Proposto | Aceito | Substituído por ADR-{X}

## Contexto
{Por que essa decisão precisa ser tomada agora}

## Opções consideradas
1. {Opção A} — prós: X. contras: Y.
2. {Opção B} — prós: X. contras: Y.

## Decisão
{Opção escolhida e motivo principal}

## Consequências
{O que fica mais fácil, o que fica mais difícil, dívida técnica gerada}
```

**Onde:** `docs/adr/ADR-{N}-{slug}.md`. Linkar no PR que implementa a decisão.

---

## DORA Metrics — Review Semanal

| Métrica | Meta | Como medir |
|---------|------|-----------|
| Deployment Frequency | ≥ 2x/semana | Vercel dashboard |
| Lead Time for Changes | Mediana < 2 dias | GitHub: PR open → merge |
| Change Failure Rate | < 10% | Rollbacks + hotfixes / total deploys |
| MTTR | < 2h | Sentry: tempo detecção → resolução |
| PR Cycle Time | < 24h mediana | GitHub PR analytics |

Registrar num doc simples (Notion ou `.claude/ops/dora.md`) semanalmente. Tendência importa mais que número absoluto.

---

## Ferramentas (Stack BR padrão)

| Uso | Ferramenta | Por quê |
|-----|-----------|---------|
| Tarefas/Sprint | Linear | Dev-focused, integração GitHub nativa |
| Docs/Wiki | Notion | Padrão BR startups, fácil para não-devs |
| Comunicação | Discord ou Slack | Discord mais comum em equipes dev BR pequenas |
| Monitoramento | Sentry + Vercel Speed Insights | Gratuito para começar |
| Observabilidade | Vercel logs + Supabase logs | Nativo da stack |

---

## Erros Comuns de Primeiro Tech Lead (BR)

1. **Fazer tudo sozinho** — "faço mais rápido eu mesmo" mata o crescimento da equipe
2. **Não negociar o escopo do cargo com o fundador** — definir explicitamente: o que André decide, o que escala para Joaquim
3. **Não escrever ADRs** — conhecimento fica tribal, sai a pessoa, vai o contexto
4. **Abrir exceções para o DoD** — "uma vez tá ok" vira padrão em 2 sprints
5. **Ignorar dívida técnica** — reservar pelo menos 10% de cada sprint para refactor
6. **Sprint 100% planejado** — sem buffer para emergências, sempre estoura

---

## Onboarding de Novo Dev (checklist)

- [ ] Acesso ao repositório GitHub + regras de branch
- [ ] Acesso ao projeto Supabase (viewer ou developer)
- [ ] Acesso ao Vercel (viewer)
- [ ] `.env.local` configurado com todas as variáveis
- [ ] `npm install && npm run dev` funcionando localmente
- [ ] Leu `docs/adr/` (todos os ADRs existentes)
- [ ] Primeiro PR pequeno e revisado em 48h (onboarding PR)
- [ ] Apresentado para o fundador e restante do time

---

## Fontes
- DORA State of DevOps Report 2024
- Rocketseat — Liderança Técnica (YouTube BR)
- https://liderancatecnica.com.br
- LinearB — Engineering Metrics Guide
