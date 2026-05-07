# Operacional — Handoffs, Fluxos Criticos e Checklist de Validacao

## Por que handoffs sao o ponto critico da Triforce Auto

O fluxo mais critico da Triforce e o de producao de LP e carrossel, que envolve 3-6 especialistas em sequencia. Cada ponto de transicao e um ponto potencial de perda de contexto. O Orquestrador e o responsavel por garantir que o pacote de handoff existe e esta completo antes de liberar o proximo passo.

Benchmark de mercado (MTM Blog, 2026): handoffs ruins custam 20-30% do tempo total de um projeto em agencias criativas.

---

## Estrutura do pacote de handoff padrao

Todo handoff entre especialistas deve conter os 4 blocos abaixo:

### Bloco 1 — O que foi feito
- Resumo do trabalho entregue
- Decisoes tomadas e por que
- O que foi descartado e por que

### Bloco 2 — O que o proximo especialista precisa fazer
- Tarefa especifica e escopo exato
- Inputs disponiveis (links, arquivos, referencias)
- Restricoes que devem ser respeitadas

### Bloco 3 — Perguntas em aberto
- O que o emissor nao sabe e o receptor pode precisar resolver
- Alertas sobre pontos de atencao

### Bloco 4 — Criterio de conclusao
- Como o receptor sabe que terminou a sua parte
- Quem recebe o proximo handoff e quando

**Nota de implementacao (validacao estrategica 2026):** Para a transicao Vitoria → Felipe (design para dev), adicionar um video curto de walkthrough do Figma (Loom, max 5 min) como complemento ao pacote escrito. Felipe nao precisa de reuniao — apenas de um walkthrough visual para handoffs de LP complexas. Incorporar como opcao no checklist para esse passo especifico.

---

## Fluxos criticos da Triforce Auto

### Fluxo LP completa (6 handoffs)

```
Rafael (pesquisa ICP + referencias)
  → [HANDOFF 1 — Rafael para Mateus]
Mateus (copy: headline, body, CTA)
  → [HANDOFF 2 — Mateus para Vitoria]
Vitoria (assets visuais da LP)
  → [HANDOFF 3 — Vitoria para Felipe | opcional: Loom walkthrough Figma]
Felipe (build no React/Vercel)
  → [HANDOFF 4 — Felipe para Marcelo]
Marcelo (revisao de codigo)
  → [HANDOFF 5 — Marcelo para Bruno]
Bruno (revisao visual final)
  → ENTREGA
```

**O que o Orquestrador garante em cada handoff do fluxo LP:**

HANDOFF 1 (Rafael → Mateus):
- Pesquisa do Rafael tem min 3 fontes identificadas
- Output foi revisado e editado (nao raw de IA)
- Formato estruturado como insumo de copy (nao apenas descricao)
- ICP definido com clareza suficiente para Mateus escrever sem adivinhar

HANDOFF 2 (Mateus → Vitoria):
- Copy completo aprovado pelo Orquestrador (7 campos do brief atendidos)
- Tom de voz documentado para referencia visual
- Dimensoes e restricoes tecnicas de formato comunicadas
- Estrutura de secoes da LP clara (quantas dobras, o que vai em cada uma)

HANDOFF 3 (Vitoria → Felipe):
- Assets exportados nos formatos corretos e nas dimensoes certas
- Especificacoes de fonte, cor (hex), espacamento documentadas
- Responsividade: variacoes mobile comunicadas ou referenciadas
- Loom opcional: walkthrough do Figma mostrando comportamentos esperados

HANDOFF 4 (Felipe → Marcelo):
- Codigo entregue em branch especifica ou URL de preview
- Lista de funcionalidades implementadas
- Pontos de atencao ou areas que precisam de revisao mais cuidadosa
- Acesso ou instrucoes para rodar o ambiente

HANDOFF 5 (Marcelo → Bruno):
- Issues resolvidos confirmados
- Codigo passou no build sem erros
- URL de preview ativa e funcionando
- Felipe confirmou correcoes aplicadas

---

### Fluxo carrossel Instagram (4 handoffs)

```
Rafael (tema, referencias visuais, dados)
  → [HANDOFF 1 — Rafael para Mateus]
Mateus (copy dos slides)
  → [HANDOFF 2 — Mateus para Vitoria]
Vitoria (design dos slides)
  → [HANDOFF 3 — Vitoria para Bruno]
Bruno (revisao design)
  → [HANDOFF 4 — Bruno para Larissa]
Larissa (publicacao e legenda)
  → PUBLICADO
```

**O que o Orquestrador garante em cada handoff do fluxo carrossel:**

HANDOFF 1 (Rafael → Mateus):
- Tema/mensagem central definidos
- Dados ou insights especificos que devem aparecer nos slides
- Referencias visuais selecionadas (referencia para Vitoria via Mateus ou diretamente)
- Numero de slides definido (5, 7 ou 10)

HANDOFF 2 (Mateus → Vitoria):
- Copy de cada slide entregue (texto exato por slide, na ordem)
- CTA do ultimo slide definido e aprovado
- Referencia visual ou mood board (se existir)
- Restricoes de tamanho de texto por slide comunicadas (max ~40 palavras)

HANDOFF 3 (Vitoria → Bruno):
- PNGs exportados nos formatos corretos (1080x1080 para feed, 1080x1920 para reels cover)
- Slides em ordem numerada
- Paleta usada documentada
- Qualquer decisao de design que se desvia do padrao da marca comunicada

HANDOFF 4 (Bruno → Larissa):
- Aprovacao explicitando o que foi validado (nao apenas "aprovado")
- Feedback corrigido confirmado se houve solicitacao de mudanca
- Arquivos finais em pasta acessivel para Larissa
- CTA e legenda sugerida (ou confirmada com Mateus)

---

### Fluxo prospeccao outbound

```
Rafael (pesquisa ICP + lista de leads potenciais)
  → [HANDOFF 1 — Rafael para Mateus]
Mateus (copy de DM / abordagem)
  → [HANDOFF 2 — Mateus para Caio]
Caio (execucao de outbound)
  → LEADS QUALIFICADOS
```

**O que o Orquestrador garante:**

HANDOFF 1 (Rafael → Mateus):
- ICP definido com sinais de compra especificos
- Lista de leads ja qualificados com contexto por lead
- Insights de nicho que devem aparecer na abordagem

HANDOFF 2 (Mateus → Caio):
- Copy de DM aprovado (sem travessao, linguagem do cliente-alvo)
- Variacoes de mensagem se necessario (abertura fria / follow-up)
- Instrucoes de personalizacao por tipo de negocio
- CTA claro (qual acao o lead deve tomar)

---

## Checklist de validacao de handoff

O Orquestrador valida este checklist ANTES de notificar o proximo especialista:

- [ ] O receptor tem todos os arquivos/links necessarios?
- [ ] As decisoes tomadas na etapa anterior estao documentadas?
- [ ] O escopo do receptor esta claro (o que fazer E o que nao fazer)?
- [ ] O criterio de conclusao da proxima etapa esta definido?
- [ ] O receptor esta disponivel (verificar mapa de capacidade)?
- [ ] Ha perguntas em aberto que precisam ser respondidas antes de comecar?
- [ ] O emissor confirmou que sua entrega esta completa (nao "quase pronta")?
- [ ] Para handoff Vitoria → Felipe: foi incluido Loom walkthrough para LP complexa?

---

## Red flags de handoff mal gerenciado

- Receptor faz perguntas que ja foram respondidas no trabalho anterior: pacote de handoff incompleto
- Felipe comeca a buildar antes do copy estar revisado: handoff liberado cedo demais
- Bruno revisa design sem saber o que a LP precisa converter: contexto de negocio nao foi propagado
- Larissa publica sem saber qual CTA foi acordado: handoff de Mateus nao incluiu intencao do texto
- Retrabalho de Felipe porque o design mudou apos o build: handoffs saindo de ordem
- Vitoria comeca design sem copy finalizado do Mateus: dependencia ignorada
- Mateus escreve copy sem pesquisa de ICP do Rafael: ordem invertida

---

## Mapa de capacidade semanal

O Orquestrador mantem uma tabela com 3 colunas por especialista, atualizada ao inicio de cada dia util:

| Especialista | Status atual | Proximo disponivel |
|---|---|---|
| Caio | Em prospeccao ativa (lista X) | [data] |
| Mateus | Escrevendo LP Barbearia | [data] |
| Felipe | Build LP em andamento | [data] |
| Vitoria | Carrossel 3 slides faltando | [hora] |
| Larissa | Agendamento da semana feito | Disponivel |
| Rafael | Pesquisando nicho Y | [hora] |
| Marcelo | Aguardando Felipe | Disponivel |
| Bruno | Aguardando Vitoria | Disponivel |

**Como atualizar:** via mensagem rapida no WhatsApp no inicio do dia: "Oi [nome], o que voce tem em andamento hoje?"

### Gargalos criticos a monitorar

- **Felipe bloqueado:** para toda producao tecnica. Prioridade maxima para desbloquear.
- **Vitoria bloqueada:** para carrosseis e assets de LP. Dependencia direta de Larissa e Bruno.
- **Mateus bloqueado:** copy e o input de quase tudo. LP sem copy = Felipe nao tem o que buildar.
- **Rafael bloqueado:** sem pesquisa, Mateus escreve no escuro, Caio prospecta sem ICP.

### Sinais de sobrecarga por especialista

**Felipe (Dev):**
- Demorando mais de 24h para responder duvidas tecnicas simples
- Commits com erros basicos que nao sao dele
- Pedindo prazo estendido para tarefas que normalmente faria rapido

**Vitoria (Designer):**
- Designs entregues com menos cuidado no detalhe
- Reutilizando elementos sem adaptar ao contexto
- Silencio prolongado (nao responde checks de status)

**Mateus (Copy):**
- Copies repetindo estruturas sem adaptar ao cliente
- Entregas atrasadas sem aviso previo
- Pedindo mais tempo para tarefas de escopo conhecido

**Rafael (Curador IA):**
- Pesquisas rasas, sem fontes solidas
- Saindo do escopo e tentando executar em vez de curar
- Outputs de IA nao revisados antes de enviar

**Caio (Prospector):**
- Leads com qualidade caindo (ICP mal qualificado)
- Diminuindo cadencia de abordagens sem comunicar
- Respondendo leads existentes com atraso

### Acao quando detecta gargalo

1. Identificar se o gargalo e de volume (muitas tarefas) ou de bloqueio (esperando input de outro)
2. Se volume: redistribuir ou atrasar tarefas de menor prioridade estrategica
3. Se bloqueio: resolver o bloqueio diretamente antes de qualquer outra acao
4. Nunca empilhar nova tarefa em especialista ja no limite sem remover uma tarefa anterior

---

## Resolucao de conflito entre especialistas

### Tipos de conflito mais comuns na Triforce Auto

**Tipo 1 — Conflito de prioridade de recurso**
Duas tarefas competem pelo mesmo especialista ao mesmo tempo.
Ex: Caio fechou lead e quer LP em 48h, mas Felipe ja esta buildando LP de outro cliente.

Resolucao:
1. Verificar qual tarefa tem maior impacto direto em receita (ICE score)
2. Verificar se ha flexibilidade de prazo em uma das tarefas
3. Se nenhuma pode ceder: escalar ao fundador com recomendacao clara ("recomendo priorizar X porque Y")

**Tipo 2 — Conflito de handoff (expectativas diferentes)**
Um especialista entregou o que entendia como certo; o receptor esperava algo diferente.
Ex: Mateus entregou copy "completo" mas Felipe diz que falta copy para o formulario.

Resolucao:
1. Verificar o brief original: o escopo estava claro?
2. Se escopo estava vago: responsabilidade do Orquestrador (corrigir o brief template)
3. Se escopo estava claro mas o especialista interpretou errado: devolver com especificacao exata do que falta
4. Nunca transformar em conflito pessoal — e sempre sobre o processo, nao sobre a pessoa

**Tipo 3 — Conflito de qualidade**
Bruno ou Marcelo reprovam trabalho que o especialista acredita estar ok.

Resolucao:
1. Verificar se os criterios de qualidade foram comunicados antes da entrega
2. Se criterios existem: Bruno/Marcelo tem autoridade — a revisao prevalece
3. Se criterios nao existiam: oportunidade de documentar o criterio para evitar repeticao
4. Orquestrador media o alinhamento, nao toma partido estetico

### Framework rapido de resolucao (5 minutos)

1. Ouvir as duas partes separadamente (2 min cada)
2. Identificar: e conflito de recursos, expectativas ou qualidade?
3. Aplicar criterio objetivo (impacto em receita / escopo documentado / criterio de qualidade)
4. Comunicar a decisao com clareza e razao
5. Documentar o padrao para evitar repeticao

### Red flags de conflito mal gerenciado

- Conflito dura mais de 24h sem resolucao: Orquestrador nao agiu
- Especialistas deixam de comunicar entre si e passam pelo Orquestrador para tudo: dependencia excessiva
- O mesmo tipo de conflito ocorre mais de 2x: causa raiz nao foi endereçada (provavelmente um SOP faltando)

---

## SOPs prioritarios para criar

1. Fluxo completo de producao de LP (Rafael → Mateus → Vitoria → Felipe → Marcelo → Bruno)
2. Fluxo de producao de carrossel Instagram (Rafael → Mateus → Vitoria → Bruno → Larissa)
3. Fluxo de prospeccao (Rafael → Caio → Mateus copy DM)
4. Checklist de publicacao de LP (testes antes de ir ao ar)
5. Onboarding de novo cliente (o que coletar no primeiro contato)

### Estrutura minima de SOP

```
NOME DO PROCESSO: [ex: Producao de LP completa]
VERSAO: [data da ultima atualizacao]
RESPONSAVEIS: [quem executa cada etapa]

ETAPAS:
1. [acao] — [responsavel] — [SLA]
2. ...

INPUTS NECESSARIOS:
- [o que precisa estar pronto antes de comecar]

OUTPUTS ESPERADOS:
- [o que deve estar pronto ao terminar]

CHECKLIST DE QUALIDADE:
- [ ] [criterio 1]
- [ ] [criterio 2]

PONTOS DE ATENCAO:
- [o que costuma dar errado e como evitar]
```

**Ferramenta para captura automatica de SOP:** Scribe (extensao gratuita de Chrome, ativa em 2026) — captura o processo automaticamente enquanto o usuario executa. Alternativa: Notion AI com templates pre-formatados.

**Como garantir que os SOPs sao usados:**
- Linkar o SOP no card do Kanban quando a tarefa comeca
- Incluir o checklist de qualidade como parte do handoff
- Revisar o SOP sempre que uma entrega gerar retrabalho evitavel
