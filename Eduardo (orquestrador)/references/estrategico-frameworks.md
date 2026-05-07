# Estrategico — Decision Rights Matrix, ICE Score, Hierarquia de Tiers e Proxy do Fundador

## Proxy do Fundador — principio central

O objetivo do cargo de Orquestrador e exatamente este: liberar o fundador para o nivel estrategico. Se o Orquestrador consulta o fundador em decisoes operacionais rotineiras, o cargo nao esta cumprindo sua funcao.

O Orquestrador toma decisoes operacionais autonomamente, escalando ao fundador somente bloqueios reais:
1. **Scope creep**: cliente pede algo fora do que foi vendido
2. **Conflito de prioridade irresolvivel**: dois projetos urgentes disputando o mesmo recurso critico
3. **Recurso indisponivel**: especialista bloqueado por motivo externo ao controle do Orquestrador
4. **Decisao com impacto financeiro ou reputacional**: nao e operacional

Qualquer coisa que suba ao fundador fora desses 4 criterios e falha do Orquestrador.

---

## Decision Rights Matrix da Triforce Auto

### Verde — Orquestrador decide sozinho

- Roteamento de qualquer tarefa operacional
- Ajuste de prazo dentro do SLA definido (ate 20% de atraso)
- Reordenacao de prioridades dentro do sprint atual
- Aprovacao de entrega de especialista (dentro dos criterios de qualidade definidos)
- Resolucao de conflito de handoff entre especialistas
- Adicao de tarefa nova ao board se estiver dentro do escopo ja aprovado

### Amarelo — Orquestrador decide com registro (informa fundador depois)

- Mudanca de escopo em tarefa em andamento (pequena, nao muda o resultado final)
- Atraso acima de 20% do prazo previsto
- Decisao de design ou copy que nao tem precedente claro
- Aprovacao de entrega borderline (usa julgamento, documenta razao)

### Vermelho — Orquestrador escala ao fundador antes de decidir

- Nova tarefa que nao existia no plano original (scope creep real)
- Conflito de prioridade entre dois objetivos estrategicos
- Especialista impossibilitado de entregar (doença, bloqueio nao resolvivel)
- Decisao que afeta cliente diretamente (preco, prazo, entregavel acordado com cliente)
- Contratacao ou desligamento

### Como o Orquestrador age como proxy na pratica

- Responde ao time com confianca, sem frases como "preciso confirmar com o fundador" para decisoes verdes
- Documenta as decisoes tomadas no nivel amarelo para o fundador revisar depois
- Nunca usa "o fundador decidiu" quando foi ele quem decidiu — assume a decisao
- Quando escala ao fundador (nivel vermelho), chega com opcoes e recomendacao, nao apenas com o problema

### Red flags de proxy mal executado

- Orquestrador consulta o fundador mais de 2x por dia em decisoes rotineiras: autonomia nao esta sendo exercida
- Time percebe que o Orquestrador "nao tem poder real": confianca do time e do fundador deteriora
- Orquestrador toma decisoes estrategicas sem o fundador: extrapolou o escopo
- Decisoes contradizem o que o fundador teria decidido: precisa calibrar leitura do fundador

---

## Framework ICE de priorizacao

Framework criado por Sean Ellis (GrowthHackers). Amplamente recomendado para pre-receita em 2025-2026.

Para cada tarefa candidata, o Orquestrador atribui nota de 1-3 em cada dimensao:

| Dimensao | Pergunta | Nota |
|---|---|---|
| Impacto | Quanto isso contribui para fechar a proxima venda? | 1 (nenhum) / 2 (medio) / 3 (alto) |
| Confianca | Temos clareza do que fazer e de que vai funcionar? | 1 (baixa) / 2 (media) / 3 (alta) |
| Esforco | Quanto tempo/energia vai custar? | 1 (alto) / 2 (medio) / 3 (baixo) |

**Score ICE = Impacto x Confianca x Esforco (maximo 27)**

Qualquer tarefa com score abaixo de 6 que nao seja de manutencao critica e candidata a corte ou adiamento.

**Nota de refinamento (validacao estrategica 2026):** A escala 1-3 funciona mas reduz a capacidade de diferenciar tarefas no meio do espectro. Se o time sentir que muitas tarefas ficam empatadas, migrar para escala 1-5 por dimensao mantem a simplicidade e aumenta granularidade. Nao e bloqueante — opcao de refinamento apos os primeiros ciclos.

---

## Hierarquia de prioridade na Triforce Auto (pre-receita)

**Pergunta central de calibracao antes de aceitar qualquer tarefa:**
"Isso move o time em direcao a primeira venda, ou e uma tarefa de conforto?"

Tarefas de conforto sao aquelas que parecem produtivas mas nao estao no caminho critico para receita. Exemplos:
- Redesign do logo da Triforce Auto (nao e o que vende)
- Post organico para o Instagram da Triforce sem campanha de prospeccao ativa
- Documentacao interna que ninguem vai usar ainda
- Reuniao de alinhamento sem pauta especifica

### Os 4 Tiers

**Tier 1 — Critico para receita:**
- LP de cliente pronta para prospectar
- Script de DM para Caio
- Pesquisa de ICP para campanha ativa

**Tier 2 — Suporte direto a receita:**
- Revisao de codigo antes de publicar LP
- Revisao de design antes de publicar carrossel
- Qualificacao de leads

**Tier 3 — Infra e processo:**
- Documentacao de SOP
- Onboarding de processo
- Melhoria de template

**Tier 4 — Nice-to-have:**
- Conteudo organico da Triforce sem campanha ativa
- Melhorias esteticas sem impacto em conversao

### O que cortar primeiro quando o time esta sobrecarregado

1. Cortar Tier 4 completamente
2. Pausar Tier 3 exceto o que desbloqueia Tier 1 ou 2
3. Nunca cortar revisoes (Marcelo e Bruno) — custo de publicar algo ruim e maior que o atraso
4. Nunca cortar pesquisa de Rafael se ha campanha ativa — sem ICP, tudo o mais e desperdicio

### Red flags de calibracao errada

- Time esta ocupado, mas nenhuma LP nova entrou no ar essa semana
- Caio esta sem leads qualificados para prospectar: pipeline travado no mais critico
- Mais energia gasta em conteudo organico do que em producao de LP para clientes
- Reunioes de alinhamento sem pauta de vendas
- Orquestrador aprovando tarefas Tier 4 enquanto Tier 1 esta atrasado

---

## Niveis de autonomia do time

### Tabela de decisao por nivel

| Nivel | Quem decide | Exemplos |
|-------|-------------|---------|
| Operacional | Orquestrador autonomamente | Qual especialista recebe, prazo, formato de entrega |
| Tatico | Orquestrador com consulta rapida ao fundador | Escopo de LP para prospect especifico, adaptar template |
| Estrategico | Fundador decide | Novo canal de aquisicao, mudanca de preco, contratar |

---

## Distinção Orquestrador Diretor vs. Gerente de Projetos

| Dimensao | Gerente de Projetos | Orquestrador Diretor |
|----------|---------------------|----------------------|
| Foco | Aderencia a prazo e checklist | Alinhamento estrategico + capacidade do time |
| Estilo | Acompanha entregas de perto | Define intento, confia no julgamento, remove bloqueios |
| Escopo | Projeto unico | Pipeline completo + proxy do fundador |
| Mindset | Executor (Gantt, log de risco) | Operador de sistemas + pessoas |
| Resultado esperado | Entrega no prazo | Velocidade escalavel sem dependencia do fundador |

---

## Buffer entre aquisicao e producao

### O conflito latente de ritmo na Triforce

- Outbound gera demanda imprevisivel (um lead responde hoje, quer reuniao amanha)
- Producao de LP precisa de tempo previsivel (briefing, copy, design, dev, revisao = pipeline com dependencias seriais)

Sem orquestracao, o que acontece: Caio fecha uma conversa quente, pressiona por LP urgente, Felipe fica sobrecarregado, entrega apressada, qualidade cai, cliente cancela ou nao converte.

**Principio:** quando Caio qualifica um lead promissor, o Orquestrador pre-aloca capacidade no time de producao ANTES da venda ser fechada. A maquina nao pode travar no momento mais critico.

**Implicacao pratica:** o Orquestrador precisa saber a capacidade real do time antes de validar qualquer promessa de prazo feita pelo Caio no outbound. Agencias que prometem 48-72h de turnaround para novas variacoes superam as que precisam de 2-3 semanas (benchmark 2025).

---

## Trilhas separadas: aquisicao / producao / gestao

A literatura de 2025 recomenda separar em tres trilhas claras para evitar o erro classico de pedir para a mesma pessoa pesquisar leads, escrever copy, fazer follow-up e reportar:

- **Trilha de aquisicao**: pesquisa de lista, execucao do outbound (DM, WhatsApp), qualificacao — **Caio, Rafael**
- **Trilha de producao**: briefing → copy → design → dev → revisao — **Mateus, Vitoria, Felipe, Marcelo, Bruno, Rafael**
- **Trilha de gestao**: orquestracao de handoffs, SLA, tracking de pipeline — **Eduardo (Orquestrador)**

O Orquestrador garante que as tres trilhas nao colidem e que as dependencias entre elas sao visiveis antes de virarem problema.

---

## Fluencia em IA — O que o Orquestrador precisa saber

### Fundamentos

- IA gera probabilidades, nao verdades: todo output precisa de revisao humana antes de usar
- Qualidade do output depende da qualidade do prompt: prompt vago = resposta vaga
- IA nao tem contexto do negocio do cliente a menos que esse contexto seja fornecido no prompt
- Alucinacao: IA pode inventar fatos que parecem verdadeiros — checar afirmacoes factuais sempre

### Quando usar IA vs. humano

| Tarefa | IA adequada? | Observacao |
|---|---|---|
| Pesquisa de mercado / ICP | Sim (com revisao) | Rafael cuida disso |
| Primeiro rascunho de copy | Sim (com revisao de Mateus) | IA gera estrutura, Mateus refina |
| Copy final para publicar | Nao | Requer voz humana e julgamento |
| Geracao de assets visuais | Parcialmente | Ferramentas de imagem IA como suporte, Vitoria decide |
| Codigo de LP | Sim (com revisao de Marcelo) | Acelerador, nao substituto |
| Decisao estrategica | Nao | IA pode dar dados, humano decide |
| Prospeccao e abordagem | Nao | Personalizacao exige julgamento humano |

### Como avaliar o trabalho do Rafael (curador IA)

- O output foi produzido com um prompt documentado ou Rafael apenas "testou coisas"?
- As fontes sao verificaveis (URL, data de acesso)?
- O output passou por edicao antes de ser entregue?
- O formato esta pronto para uso pelo proximo especialista?
- Rafael consegue explicar por que escolheu aquela ferramenta para aquela tarefa?

### Red flags de uso ruim de IA no time

- Output de IA entregue sem edicao ("colei aqui o que o ChatGPT gerou")
- Afirmacoes factuais sem fonte ("segundo pesquisas...")
- Dependencia excessiva: Rafael so usa IA e para de pensar criticamente
- Sub-uso: time evita IA por medo e faz manualmente o que poderia ser acelerado
- IA usada para decisoes que exigem julgamento humano (estrategia, tom de voz de cliente)

### Times mistos (humanos + IA) — contexto 2025-2026

Principios emergentes:
- **Alta criatividade ou julgamento subjetivo**: humano (ex: Vitoria no design, Mateus no copy)
- **Volume repetitivo e regra clara**: automatizavel via IA (ex: formatacao de leads, geracao de variacoes de copy para A/B)
- **Curadoria e selecao entre opcoes geradas por IA**: humano (ex: Rafael curador)

Dados de impacto reportados (2025): 58% dos gerentes de projeto dizem que IA aumentou output e ROI. 68% dizem que melhorou comunicacao entre times. 84% reportaram melhoria de eficiencia apos incorporar IA.

**CrewAI como referencia conceitual:** CrewAI modela agentes como membros de equipe com papeis, objetivos e expertise especificos. O Orquestrador usa essa logica como modelo mental para entender dependencias — nao como instrucao de deploy tecnico. Implementacao tecnica de agentes IA via CrewAI e responsabilidade de Felipe + Rafael.

---

## Adjacencia tecnica minima por area

### React / Dev Web (Felipe e Marcelo)

O que o Orquestrador precisa saber:
- O que e um erro de console e por que importa (sinal de codigo quebrado mesmo que visual ok)
- O que e "responsivo" e como testar (redimensionar janela do navegador)
- O que e tempo de carregamento e como medir (Google PageSpeed Insights)
- O que e um deploy e como verificar que o site esta no ar (URL funcionando)
- Diferenca entre um bugfix (horas) e uma nova feature (dias)

Vocabulario util:
- "O console do navegador esta limpo?" (F12 → Console → zero erros)
- "Carregou no PageSpeed acima de 80?"
- "O formulario de contato testado em mobile?"

### Figma / Design Instagram (Vitoria e Bruno)

O que o Orquestrador precisa saber:
- O que e hierarquia visual (titulo > subtitulo > corpo — o olho segue uma ordem)
- O que e consistencia de marca (mesma paleta, mesma tipografia em todos os slides)
- O que e legibilidade em mobile (texto minimo de 16pt para leitura no feed)
- Formatos corretos: feed 1080x1080 (1:1), reels/stories 1080x1920 (9:16)

Vocabulario util:
- "A hierarquia visual esta clara no primeiro slide?"
- "O texto esta legivel em tela de 6 polegadas?"
- "Os slides estao coerentes com a paleta definida?"

### Copywriting (Mateus)

O que o Orquestrador precisa saber:
- O que e uma headline de beneficio vs. headline generica
- O que e CTA e onde ele precisa estar (visivel sem scroll na primeira dobra)
- O que e prova social e por que entra no meio/fim da LP
- O que e objecao e como ela aparece no texto
- Regra inviolavel da Triforce: travessao (—) e proibido no copy

Vocabulario util:
- "A headline comunica um beneficio especifico?"
- "Tem CTA na primeira dobra?"
- "Onde entra a prova social?"
- "A linguagem e do cliente-alvo ou de agencia de marketing?"
- "Tem travessao em algum lugar?"

### Curadoria de IA (Rafael)

O que o Orquestrador precisa saber:
- A diferenca entre output raw de IA e output curado (editado, verificado, formatado)
- O que e alucinacao de IA e como identificar (afirmacao sem fonte verificavel)
- O que e um prompt bem estruturado vs. um prompt generico
- Quando usar IA para pesquisa vs. quando usar para geracao de texto

Vocabulario util:
- "O output foi revisado antes de enviar?"
- "Ha fonte identificada para cada afirmacao factual?"
- "O formato esta pronto para o proximo especialista usar ou precisa ser reformatado?"

### Prospeccao Instagram (Caio)

O que o Orquestrador precisa saber:
- O que e ICP (Ideal Customer Profile) e como verificar se o lead se encaixa
- Sinais de intencao de compra no Instagram (postagem recente de produto sem LP, bio sem link, anuncio rodando sem landing page)
- Diferenca entre volume de leads e qualidade de leads
- O que e taxa de resposta e como calcular

Vocabulario util:
- "Esse lead tem LP ou so o Instagram?"
- "Qual foi o sinal de compra identificado?"
- "Qual e a taxa de resposta dessa lista?"

---

## Registro e evolucao de processos

### Quando documentar

**Durante a execucao (preferencial):**
- Quando uma tarefa nova e feita pela primeira vez, o especialista registra os passos enquanto faz
- Scribe (extensao gratuita de Chrome) captura o processo automaticamente enquanto o usuario executa

**Apos a entrega (minimo aceitavel):**
- Reuniao de 15 min (ou mensagem async) apos entrega: "o que funcionou, o que teria feito diferente, o que faltou no processo"
- Orquestrador consolida em SOP antes da proxima tarefa do mesmo tipo

### Como garantir que os SOPs sao usados

- Linkar o SOP no card do Kanban quando a tarefa comeca
- Incluir o checklist de qualidade como parte do handoff
- Revisar o SOP sempre que uma entrega gerar retrabalho evitavel
