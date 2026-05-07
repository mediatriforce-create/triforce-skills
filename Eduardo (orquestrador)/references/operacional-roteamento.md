# Operacional — Roteamento, Brief, SLAs e Fluxos Criticos

## Sistema de classificacao: 3 perguntas-gatilho

Ao receber qualquer pedido, o Orquestrador responde internamente:

### Pergunta 1 — Qual e a competencia central exigida?

- Gerar leads / prospectar: **Caio**
- Escrever texto persuasivo / CTA / copy de LP: **Mateus**
- Construir ou alterar pagina/sistema: **Felipe**
- Criar visual para Instagram / carrossel / story: **Vitoria**
- Publicar, agendar, engajar nas redes: **Larissa**
- Pesquisar, curar, usar ferramentas IA: **Rafael**
- Revisar codigo antes de publicar: **Marcelo**
- Revisar design antes de publicar: **Bruno**
- Tarefa envolve multiplos especialistas: mapear handoff completo (ver operacional-handoffs.md)

### Pergunta 2 — Qual e a relacao urgencia x complexidade?

- Alta urgencia + baixa complexidade: rotear direto, brief curto
- Alta urgencia + alta complexidade: rotear com brief detalhado E sinalizar risco de prazo
- Baixa urgencia + alta complexidade: rotear com prazo estendido, espaco para iteracao
- Baixa urgencia + baixa complexidade: batch com outras tarefas do mesmo especialista

### Pergunta 3 — Ha dependencias de handoff?

- Se a tarefa exige output de um especialista como input de outro (ex: Rafael pesquisa → Mateus escreve → Felipe implementa), o Orquestrador deve mapear a sequencia ANTES de delegar o primeiro passo.
- Nunca delegar passo 1 sem garantir que o receptor do passo 2 esta disponivel.

---

## Matriz de roteamento por tipo de entrega

| Tipo de Pedido | Especialista Primario | Sequencia Completa de Dependencias |
|---|---|---|
| Producao de LP | Felipe (build) | Rafael (ICP + referencias) → Mateus (copy: headline, body, CTA) → Vitoria (assets visuais) → Felipe (build React/Vercel) → Marcelo (revisao codigo) → Bruno (revisao visual final) |
| Carrossel Instagram | Vitoria (design) | Rafael (tema, referencias visuais, dados) → Mateus (copy dos slides) → Vitoria (design dos slides) → Bruno (revisao design) → Larissa (publicacao e legenda) |
| Campanha de prospeccao | Caio (leads) | Rafael (ICP/pesquisa de nicho) → Mateus (copy DM) → Caio (execucao de outbound) |
| Automacao / IA | Rafael (curadoria) | Felipe (implementacao tecnica se necessario) |
| Revisao de codigo | Marcelo | Felipe (entrega) → Marcelo (revisao) → Felipe (correcao) → Marcelo (aprovacao final) |
| Revisao de design | Bruno | Vitoria (entrega) → Bruno (revisao) → Vitoria (correcao se necessario) → Bruno (aprovacao final) |
| Post organico | Larissa | Mateus (copy) → Vitoria (arte) → Bruno (revisao) → Larissa (agendamento) |

---

## Red flags de roteamento errado

- Especialista entrega algo que nao era o que o fundador queria: provavelmente o pedido foi mal classificado na entrada
- Felipe recebe tarefas de design ou copy que nao sao dele: confusao entre "precisa de site" e "precisa de tudo junto"
- Caio prospectando sem copy aprovado: foi roteado antes da dependencia estar resolvida
- Mateus escreve copy sem pesquisa de ICP do Rafael: ordem de handoff invertida
- Vitoria comeca design sem copy finalizado: handoff liberado cedo demais
- Felipe comeca build antes do design aprovado por Bruno: handoffs saindo de ordem

---

## Metricas de roteamento

- Taxa de retrabalho por tarefa (ideal: menor que 20% das entregas retornam por erro de roteamento)
- Numero de tarefas que voltam por "especialista errado"
- Tempo medio entre pedido do fundador e inicio de execucao pelo especialista certo (benchmark: menos de 4h para tarefas urgentes, menos de 24h para tarefas normais)

---

## Formato padrao de brief — 7 campos

Todo brief entregue pelo Orquestrador deve conter os 7 campos abaixo. Campos em negrito sao obrigatorios; os demais sao opcionais se o contexto for obvio.

```
TAREFA: [o que precisa ser feito, em 1-2 frases]
INTENTO: [por que isso importa agora, qual resultado de negocio isso apoia]
INPUTS: [o que o especialista ja tem disponivel para comecar]
DONO: [nome do especialista responsavel pela entrega]
PRAZO: [data/hora limite ou SLA do tipo de tarefa]
CRITERIO DE SUCESSO: [como o Orquestrador vai saber que a entrega esta boa]
GATILHO DE ESCALADA: [o que justifica o especialista parar e pedir ajuda antes do prazo]
```

### Exemplo aplicado: brief para Mateus (copy de LP)

```
TAREFA: Escrever copy completo para LP do cliente Barbearia Nova Era (homepage).
INTENTO: LP vai ser o ativo central da campanha de prospeccao que o Caio vai usar essa semana. Quanto mais rapido o copy estiver pronto, mais cedo o Caio consegue converter os leads que ja estao na lista.
INPUTS: Pesquisa de ICP feita pelo Rafael (arquivo em [link]). Referencia visual da LP no Figma (link). Tom de voz: direto, sem frescura, fala com homem 25-45 anos.
DONO: Mateus
PRAZO: Quinta 18h
CRITERIO DE SUCESSO: headline clara com beneficio, CTA visivel em cada dobra, sem jargao de marketing, sem travessao.
GATILHO DE ESCALADA: Se o ICP do Rafael estiver incompleto ou contraditorio, avisar antes de escrever.
```

### Niveis de delegacao por maturidade do especialista

| Nivel | Descricao | O que o Orquestrador passa |
|---|---|---|
| 1 — Demonstracao | Especialista novo na funcao | Mostra como fazer, acompanha de perto |
| 2 — Tentativa guiada | Especialista aprendendo | Brief detalhado + checklist de entrega |
| 3 — Execucao independente | Especialista maduro na funcao | Brief de intento, criterio de sucesso, prazo |
| 4 — Ensina outros | Especialista senior (meta futura) | Apenas intento e resultado esperado |

Mapeamento atual do time Triforce:
- Felipe e Marcelo: nivel 3-4
- Caio, Vitoria, Larissa: nivel 2-3 dependendo do tipo de tarefa
- Mateus e Rafael: nivel 3 para tarefas conhecidas, nivel 2 para tarefas novas

Nota (validacao estrategica): no estagio atual da Triforce, nenhum especialista deve ser categorizado como nivel 4. O nivel 4 implica capacidade de replicar conhecimento para novos membros — relevante apenas quando a empresa comecar a contratar para as mesmas funcoes. Usar como meta de crescimento individual, nao como descricao de status atual.

### Red flags de delegacao mal feita

- Especialista entrega algo tecnicamente correto mas estrategicamente errado: intento nao foi comunicado
- Especialista faz multiplas perguntas de volta antes de comecar: inputs insuficientes ou criterio de sucesso vago
- Entrega chega no prazo mas precisa de refacao completa: criterio de sucesso nao foi definido na delegacao
- Especialista entregou mais do que pedido (gold-plating): escopo nao estava claro

---

## Tabela de SLAs internos

| Tipo de Entrega | Responsavel | SLA Padrao | SLA Urgente |
|---|---|---|---|
| Pesquisa de ICP / tema | Rafael | 24h | 4h |
| Copy de carrossel (5-10 slides) | Mateus | 24h | 8h |
| Copy de LP completa | Mateus | 48h | 24h |
| Design de carrossel (5-10 slides) | Vitoria | 48h | 24h |
| Assets visuais de LP | Vitoria | 24h | 8h |
| Build de LP (React/Vercel) | Felipe | 72h | 48h |
| Revisao de codigo | Marcelo | 24h | 4h |
| Revisao de design | Bruno | 24h | 4h |
| Lista de leads qualificados (20) | Caio | 48h | 24h |
| Publicacao de post | Larissa | 2h apos aprovacao | 1h |

### SLAs do fluxo LP (tempo total)

- Briefing → Copy: 48h
- Copy → Design: 24h apos aprovacao
- Design → Dev: 24h apos aprovacao Bruno
- Dev → Revisao Marcelo: 24h
- Revisao → Deploy: 4h

### Protocolo de gestao de SLA

**Quando o Orquestrador deve agir:**
- T-25% do prazo: check-in leve ("Oi, como ta o andamento?")
- T-10% do prazo: se nao tiver atualizacao, check-in direto
- Prazo vencido sem entrega: acionar imediatamente, entender causa, ajustar

**Categorias de descumprimento:**

| Causa | Acao do Orquestrador |
|---|---|
| Especialista sobrecarregado | Redistribuir ou priorizar — nunca empilhar mais |
| Bloqueio (aguardando input) | Resolver o bloqueio antes de qualquer outra acao |
| Tarefa subestimada no escopo | Ajustar SLA para o tipo e documentar |
| Problema pessoal / imprevisto | Redistribuir com urgencia |
| Padrao repetido (especialista sempre atrasa) | Conversa individual + revisao de workload |

**Metricas de SLA:**
- Taxa de cumprimento de SLA por especialista (meta: acima de 80%)
- Atraso medio quando SLA e descumprido (benchmark aceitavel: menos de 4h de atraso)
- Tipos de entrega com maior taxa de descumprimento (identificar gargalos sistemicos)

**Como comunicar SLA ao time sem criar resistencia:**
- Apresentar como acordo mutuo, nao como imposicao: "Qual e o prazo razoavel pra voce nesse tipo de entrega?"
- Primeiro ciclo: usar SLAs como referencia, nao como punicao
- Revisao mensal: ajustar baseado no que esta funcionando

---

## Board Kanban do Orquestrador

### Estrutura de colunas

| Coluna | O que entra | Responsavel de mover |
|---|---|---|
| A Rotear | Pedido chegou, ainda nao foi delegado | Orquestrador |
| Em Andamento | Tarefa delegada, especialista executando | Especialista |
| Em Handoff | Tarefa concluida por um especialista, aguardando proximo | Orquestrador |
| Em Revisao | Entregue para Bruno ou Marcelo revisar | Revisor |
| Concluido | Aprovado e entregue/publicado | Orquestrador |
| Bloqueado | Parado aguardando algo externo | Orquestrador |

### Color-code de risco

- Verde: no prazo, sem problemas
- Amarelo: em andamento, mas proximo do prazo
- Vermelho: atrasado ou em risco de nao entregar
- Cinza: bloqueado (aguardando input externo)
- Roxo: dependencia critica (move uma outra tarefa)

### Campos obrigatorios em cada card

- Nome da tarefa
- Especialista responsavel
- Prazo
- Cor de risco
- Dependencias (qual card precisa estar concluido antes)
- Link para o pacote de handoff (quando aplicavel)

### Rotina de manutencao do board

- **Diariamente (5 min):** mover cards que mudaram de status, atualizar cores de risco
- **Semanalmente (15 min):** revisar coluna "Em Andamento" para identificar tarefas paradas sem atualizacao
- **A cada nova tarefa:** criar card ANTES de delegar (nunca delegar sem card)

### Red flags no board

- Tarefa em "Em Andamento" ha mais de 3 dias sem atualizacao: provavel bloqueio silencioso
- Coluna "Em Handoff" acumulando cards: Orquestrador nao esta processando as transicoes
- Tarefa movida direto de "Em Andamento" para "Concluido" sem passar por "Em Revisao": revisao foi pulada
- Nenhum card em vermelho quando o time esta visivelmente atrasado: cores nao estao sendo atualizadas

**Ferramenta recomendada:** Notion (permite hospedar board + SOPs + briefs no mesmo workspace, eliminando fragmentacao de contexto). Trello funciona mas nao integra com base de SOPs.

---

## Protocolo de sintese de briefing do fundador

### 4 passos

**Passo 1 — Ouvir e registrar o pedido bruto**
Nao interromper. Deixar o fundador falar tudo. Registrar em texto (mesmo que seja um audio: transcrever).

**Passo 2 — Classificar o que esta claro vs. vago**

Categorias de informacao necessaria para qualquer tarefa:
- Resultado de negocio esperado (o que muda depois que essa tarefa for feita?)
- Cliente/nicho alvo
- Prazo real (nao o "seria bom ter" — o "precisa estar no ar ate quando")
- Tom e restricoes (o que nao pode aparecer? o que e inegociavel?)
- Formato e canal (LP? Carrossel? Post? WhatsApp?)

**Passo 3 — Fazer no maximo 3 perguntas de clarificacao**

Regra: nunca fazer mais de 3 perguntas de uma vez. Priorizar as que desbloqueiam mais decisoes downstream.

Exemplos de perguntas de alto valor:
- "Qual e o objetivo principal dessa LP — capturar lead ou vender direto?"
- "Quem e o dono do negocio? Tem algum material de identidade visual?"
- "Quando isso precisa estar no ar? Ha uma data de lancamento de campanha?"

**Passo 4 — Resumir de volta antes de delegar**
Antes de criar o brief para o especialista, confirmar com o fundador em 3-4 linhas: "Entendi que voce quer [X] para [Y], com prazo [Z], com o objetivo de [resultado]. Correto?"

### Perguntas-gatilho por tipo de tarefa

**Para LP:**
- Qual produto/servico vai na LP?
- Qual e o CTA principal? (WhatsApp / Formulario / Ligacao)
- Tem identidade visual do cliente?
- Quando precisa estar no ar?
- Vai ter trafego pago ou apenas organico?

**Para carrossel Instagram:**
- Qual e o tema/mensagem central?
- E para o perfil do cliente ou para a Triforce?
- Qual e o CTA do ultimo slide?
- Tem alguma referencia visual?
- Quando publica?

**Para campanha de prospeccao:**
- Qual nicho vai ser prospectado?
- Qual e a oferta que o Caio vai apresentar?
- Qual e o volume de abordagens esperado?

### Red flags de sintese mal feita

- Brief chega ao especialista com campo "INTENTO" vazio: o Orquestrador nao perguntou o por que
- Especialista pergunta "mas para qual cliente isso e?" depois de receber o brief: informacao basica nao foi capturada
- Tarefa entregue dentro do escopo mas fundador diz "nao era isso": o resumo de confirmacao nao foi feito
- Prazo do especialista e o prazo do fundador sao diferentes: nao houve alinhamento de expectativa
