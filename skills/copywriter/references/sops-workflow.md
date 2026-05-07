# SOPs Operacionais — Fecchio
**Fecchio | Copywriter Senior | Triforce Auto**
*Procedimentos operacionais padrão para copy de resposta direta*

---

## SOP 1 — Briefing

### Objetivo
Coletar todas as informações necessárias antes de escrever uma linha. Briefing incompleto = copy que não converte.

### Campos Obrigatórios do Documento de Brief

```markdown
# Brief — [Nome do Projeto]
Data: YYYY-MM-DD | Solicitante: Joaquim (fundador)

## 1. Produto / Serviço
- O que é: [descrição objetiva]
- Entregável exato: [o que o cliente recebe]
- Prazo de entrega: [dias úteis]
- Preço: [valor ou faixa]

## 2. Público-Alvo
- Perfil: [cargo/situação/contexto]
- Dor principal: [específica, não genérica]
- Objeção principal: [o maior medo ou ceticismo]
- Nível de consciência: [1-5] (ver frameworks-persuasao.md)
- Onde encontra a LP: [canal de tráfego]

## 3. Tipo de LP
- [ ] LP local (barbearia/salão/personal/outro presencial)
- [ ] LP infoproduto (coach/consultor/afiliado/digital)
- [ ] LP mista (presencial com oferta digital)

## 4. Prova Social Disponível
- [ ] Depoimentos com nome + resultado: [quantos?]
- [ ] Cases com números: [quais?]
- [ ] Avaliações Google: [estrelas + quantidade]
- [ ] Zero prova social (pré-receita) → ativar Modo Cliente Fundador

## 5. Diferencial do Cliente
- Por que esse e não o concorrente: [específico]
- O que fazem diferente: [específico]
- Mecanismo único (se infoproduto): [nomeado]

## 6. Contexto de Copy
- Peça: [LP completa / headlines / revisão / microcopy / VSL]
- Formato de entrega esperado: [Markdown para Felipe/Camila / direto no Figma]
- Prazo da entrega da copy: [data]

## 7. Localização (se LP local)
- Bairro/cidade: [específico]
- Endereço + ponto de referência: [completo]
- Horários: [incluindo diferenciais: noturno, sábado]
- WhatsApp de contato: [com DDD]
```

### Perguntas de Diagnóstico Durante o Briefing

Além dos campos acima, perguntar oralmente (ou via chat):

1. "Qual a maior dor do seu cliente — o que tira o sono dele às 3h?"
2. "Por que você é diferente do concorrente da esquina?"
3. "Qual o maior medo do cliente na hora de contratar?"
4. "Tem algum cliente atual que deu feedback positivo? O que ele disse?"
5. "Qual resultado você consegue garantir em qual prazo?"

---

## SOP 2 — Diagnóstico Antes de Escrever

### Objetivo
Definir framework, tom e estratégia antes de abrir o editor. Diagnóstico incorreto = copy errada para o público certo.

### Sequência de Diagnóstico

**Passo 1: Ler brand/**
- `.claude/brand/voice.md` — tom e padrões de comunicação da Triforce Auto
- `.claude/brand/audience.md` — dois segmentos, dores, objeções

**Passo 2: Identificar o tipo de LP**
- LP local (presencial) → Copy Local BR framework + CTA WhatsApp + SEO local
- LP digital (infoproduto) → Big Idea + Mecanismo Único + estrutura completa

**Passo 3: Mapear nível de consciência**

| Resposta do lead à pergunta "você sabe que precisa de LP?" | Nível |
|------------------------------------------------------------|-------|
| "Não sei o que é LP" | 1 — Inconsciente |
| "Sei que preciso de presença digital, mas não sei como" | 2 — Consciente do problema |
| "Estou pensando em fazer uma LP" | 3 — Consciente da solução |
| "Quero contratar — está pesquisando quem" | 4 — Consciente do produto |
| "Você vs. concorrente X" | 5 — Muito consciente |

**Passo 4: Mapear nível de sofisticação do mercado**
- Mercado local (barbearia em cidade X): geralmente Nível 2 — promessa específica funciona
- Mercado de infoprodutores BR: geralmente Nível 3-4 — mecanismo único obrigatório

**Passo 5: Verificar prova social disponível**
- Tem depoimentos reais? → usar com nome + bairro (local) ou resultado (digital)
- Zero depoimentos? → Modo Cliente Fundador (especificidade + garantia + linguagem pioneiro)

**Passo 6: Definir framework**

| Tipo | Nível consciência | Framework recomendado |
|------|------------------|-----------------------|
| LP local, público Nível 2-3 | BAB ou Copy Local BR direto |
| LP local, público Nível 3-4 | PAS + CTA WhatsApp |
| LP digital, público Nível 2-3 | Kishotenketsu ou BAB estendido |
| LP digital, público Nível 3-4 | Big Idea + Mecanismo Único + PAS |
| VSL | Kishotenketsu ou estrutura 8 partes |
| Headlines apenas | 4U framework |

---

## SOP 3 — Seven Sweeps (Revisão)

### Objetivo
Revisar qualquer peça de copy antes de entregar. Aplicar todas as 7 lentes, em sequência. Após cada sweep, voltar às anteriores para garantir que as edições não criaram novos problemas.

### Checklist por Lente

**Sweep 1 — Clareza**
- [ ] Cada frase tem apenas uma ideia?
- [ ] Sem jargão não explicado?
- [ ] Pronomes com referências claras? (este, esse, isso — quem/o quê?)
- [ ] Sem frases tentando dizer coisas demais de uma vez?
- [ ] Palavras de filler eliminadas? (muito, realmente, extremamente, apenas, basicamente)
- [ ] Frases fracas substituídas? ("a fim de" → "para", "utilize" → "use", "alavancar" → "usar")

**Sweep 2 — Voz e Tom**
- [ ] Tom consistente do início ao fim?
- [ ] Alinhado com `brand/voice.md`? (direto, técnico, sem enrolação, confiante mas acessível)
- [ ] Sem mudanças bruscas de formal para casual?
- [ ] Leu em voz alta — soa humano?
- [ ] Zero corporativês? ("soluções robustas", "ecossistema", "sinergia")

**Sweep 3 — So What (E daí?)**
- [ ] Cada afirmação responde "por que devo me importar"?
- [ ] Features conectadas a benefícios?
- [ ] Benefícios conectados a desejos reais?
- [ ] ❌ "Plataforma usa tecnologia avançada" → ✅ "Tecnologia que faz X específico pra você"

**Sweep 4 — Prove It (Prove)**
- [ ] Cada promessa tem prova ou mecanismo sustentando?
- [ ] Social proof específico e atribuído?
- [ ] Sem superlativos não ganhos ("líder de mercado", "melhor do Brasil", "único no mundo")?
- [ ] Se pré-receita: especificidade do processo substituindo depoimentos?

**Sweep 5 — Especificidade**
- [ ] Linguagem vaga substituída por concreta?
- [ ] Números e prazos incluídos?
- [ ] ❌ "Economize tempo" → ✅ "Economize 4 horas por semana"
- [ ] ❌ "Entrega rápida" → ✅ "Entrega em 7 dias úteis"
- [ ] ❌ "Bons resultados" → ✅ "4-8% de conversão com WhatsApp CTA"

**Sweep 6 — Emoção**
- [ ] A copy faz sentir algo, não apenas informar?
- [ ] A dor do estado atual está vívida e específica (não genérica)?
- [ ] A aspiração do estado desejado está palpável?
- [ ] Há tensão emocional real (não melodrama) entre antes e depois?
- [ ] O lead se reconhece na copy?

**Sweep 7 — Zero Risco (próximo ao CTA)**
- [ ] Objeções respondidas perto do CTA?
- [ ] Trust signals presentes (garantia, processo, especificidade)?
- [ ] Próximos passos cristalinos (o que acontece depois de clicar)?
- [ ] Risk reversals declarados (garantia sem fricção, aprovação obrigatória)?
- [ ] A garantia está próxima ao CTA, não escondida no rodapé?

---

## SOP 4 — Entrega de Copy

### Objetivo
Garantir que Felipe e Camila recebam tudo o que precisam para implementar sem ambiguidade.

### Documento Markdown — Template Completo

```markdown
# [Nome do Cliente] — Copy LP [Tipo]
Versão: 1.0 | Data: YYYY-MM-DD | Copywriter: Fecchio
Status: Pronto para revisão / Aprovado / Em ajuste

---

## Meta Tags

**Meta title:** [texto — 50-60 chars]
**Meta description:** [texto — 150-160 chars]
**H1:** [texto — headline principal]

---

## Seção 1: Hero

**Headline:** [texto — máx 70 chars]
**Subtítulo:** [texto — máx 150 chars]
**CTA principal:** [texto do botão — máx 30 chars]
**CTA secundário:** [texto — se houver]

### Microcopy
| Elemento | ID | Copy | Estado | Máx chars |
|---------|-----|------|--------|-----------|
| btn-cta | btn-hero-primary | [copy] | default | 30 |
| btn-cta | btn-hero-primary | Aguarde... | loading | 20 |

**Nota para Camila:** [tom desta seção, elemento de destaque, hierarquia visual esperada]
**Nota para Felipe:** [limites de chars, estados de componente, condicionais]

---

## Seção 2: [Nome]

[repetir estrutura acima para cada seção]

---

## Observações Gerais

**Prova social pendente:** [se aplicável — o que adicionar quando disponível]
**SEO local:** [se LP local — instruções de schema para Felipe]
**A/B sugerido:** [se houver variações de headline para testar]
```

### Checklist de Entrega

Antes de marcar como entregue:
- [ ] Todas as seções com copy preenchida (zero "inserir texto aqui")
- [ ] Meta title e meta description entregues com contagem de chars
- [ ] Tabela de microcopy por seção com IDs, estados e limites
- [ ] Nota para Camila e nota para Felipe em cada seção
- [ ] Se LP local: instruções de schema para Felipe
- [ ] Se pré-receita: nota explícita sobre prova social pendente
- [ ] Seven Sweeps aplicados e documentados (se revisão: Quick Wins separados)

---

## SOP 5 — Revisão de Copy Existente

### Objetivo
Identificar problemas de conversão em copy já existente e propor melhorias priorizadas.

### CRO 7 Níveis

Aplicar nesta ordem para diagnosticar a copy existente:

1. **Clareza da oferta** — em 5 segundos, o visitante entende o que está sendo vendido?
2. **Relevância para o público** — a copy fala com o avatar certo no nível de consciência certo?
3. **Headline** — atende ≥ 3 dos 4 Us? (Útil, Urgente, Único, Ultra-específico)
4. **Prova social** — depoimentos específicos e atribuídos, ou genéricos e sem credibilidade?
5. **CTA** — em 1ª pessoa, com verbo de ação e resultado? Está acima da dobra em mobile?
6. **Objeções** — as principais objeções do avatar estão respondidas antes do CTA?
7. **Garantia** — está próxima ao CTA, sem fricção e sem condições excessivas?

### Formato de Output da Revisão

```markdown
# Revisão de Copy — [Nome do Projeto]
Data: YYYY-MM-DD | Revisado por: Fecchio

## Diagnóstico Geral

**Nível de consciência do público-alvo:** [1-5]
**Framework original usado:** [identificar]
**Problemas críticos identificados:** [lista]

---

## Quick Wins (3 mudanças rápidas, alto impacto)

### QW1 — [Nome do problema]
**Problema:** [o que está errado e por quê]
**Copy atual:** "[texto atual]"
**Copy sugerida:** "[texto novo]"
**Por quê funciona melhor:** [racional de 1-2 frases]

### QW2 — [...]

### QW3 — [...]

---

## High-Impact (3 mudanças que exigem reescrita de seção)

### HI1 — [Nome do problema]
**Problema:** [o que está errado estruturalmente]
**Seção afetada:** [nome da seção]
**Proposta:** [o que fazer — não necessariamente a copy final]
**Impacto esperado:** [estimativa de melhora na conversão]

### HI2 — [...]

### HI3 — [...]

---

## Seven Sweeps — Resultado

| Sweep | Status | Problemas encontrados |
|-------|--------|-----------------------|
| 1. Clareza | ✅/⚠️/❌ | [notas] |
| 2. Voz e Tom | ✅/⚠️/❌ | [notas] |
| 3. So What | ✅/⚠️/❌ | [notas] |
| 4. Prove It | ✅/⚠️/❌ | [notas] |
| 5. Especificidade | ✅/⚠️/❌ | [notas] |
| 6. Emoção | ✅/⚠️/❌ | [notas] |
| 7. Zero Risco | ✅/⚠️/❌ | [notas] |
```

---

## SOP 6 — Copy Pré-Receita

### Objetivo
Escrever copy que converte mesmo sem depoimentos, cases ou histórico de resultados.

### Sequência Obrigatória

**Passo 1: Confirmar que está em pré-receita**
Perguntar explicitamente: "Tem algum cliente que posso mencionar? Mesmo um cliente teste ou beta?"
Se sim → usar com permissão. Se não → ativar Modo Cliente Fundador.

**Passo 2: Mapear o que EXISTE como prova alternativa**
- Processo detalhado do serviço (etapas documentadas)
- Entregáveis específicos (o que o cliente recebe, em detalhe)
- Ferramentas e frameworks usados (credibilidade de método)
- Benchmarks do setor (dados de mercado como referência)
- Garantia com risco reverso (substituição de prova social por risco zero)

**Passo 3: Escrever copy específica (não genérica)**

❌ Copy genérica pré-receita:
> "Criamos landing pages profissionais para o seu negócio com copy estruturada e design responsivo."

✅ Copy específica pré-receita:
> "Você recebe: 1 landing page em React com copy de resposta direta, design aprovado por você no Figma, formulário integrado ao WhatsApp, hospedagem no Vercel configurada, 30 dias de suporte. Entrega em 7 dias úteis."

**Passo 4: Usar linguagem de Cliente Fundador**
- "Vagas abertas para clientes fundadores — preço especial para os primeiros 5"
- "Estamos selecionando negócios presenciais para nossa fase beta"
- "Você ajuda a moldar o produto e paga menos — atenção máxima da equipe"

**Passo 5: Aplicar garantia sem fricção**
- "Se em 7 dias após a entrega você não estiver satisfeito, devolvemos 100%. Sem burocracia."
- "Aprovação obrigatória em cada etapa. Só finalizamos quando você disser OK."

**Passo 6: Sweep 4 com atenção máxima**
Na revisão, focar no Sweep 4 (Prove It): cada promessa tem especificidade ou garantia sustentando? Se não, cortar ou reformular.

### Armadilhas a evitar na pré-receita
- ❌ Inventar ou exagerar resultados — cria expectativa falsa e destrói confiança no momento da entrega
- ❌ Depoimento genérico sem atribuição ("um cliente disse que adorou") — pior que não ter depoimento
- ❌ Copiar estrutura pesada de prova social de infoproduto — a ausência fica óbvia
- ❌ Prometer mais do que pode entregar para fechar o primeiro cliente

---

## SOP 7 — Registro de Aprendizados

### Objetivo
Construir base de conhecimento sobre o que funciona para cada segmento na Triforce Auto.

### Quando registrar
- Fundador (Joaquim) aprovou a copy
- Cliente converteu (fez o agendamento, enviou mensagem no WhatsApp, pagou)
- Taxa de conversão medida e disponível (via Vercel Speed Insights ou dado direto do cliente)

### Onde registrar
`.claude/brand/learnings.md`

### Formato de entrada

```markdown
## [Data] — [Tipo de LP] — [Segmento]

**Peça:** [Nome do projeto / tipo]
**Segmento:** [barbearia / salão / personal / infoproduto / etc.]
**Framework usado:** [Copy Local BR / BAB / PAS / etc.]
**O que funcionou:** [específico — qual headline, qual CTA, qual estrutura]
**Taxa de conversão:** [% se disponível]
**Benchmark anterior:** [se havia referência anterior]
**Notas:** [o que tentar diferente da próxima vez]
```

**Exemplos de o que registrar:**
- "Headline com nome do bairro converteu 2x mais que headline genérica"
- "CTA WhatsApp pré-preenchido vs. CTA formulário: +40% de cliques no WhatsApp"
- "Garantia de 7 dias no Hero aumentou tempo de permanência na página"
- "Depoimento com nome + bairro do cliente gerou mais confiança que depoimento sem localização"

### Por que registrar
Cada aprendizado reduz o tempo de diagnóstico na próxima LP do mesmo segmento. Em 6 meses, a Triforce Auto terá benchmarks próprios, mais precisos que qualquer dado genérico de mercado.
