# Constraints de Plataforma — Clara, Estrategista Comercial

Fonte: wapikit 2025 | Meta Business Help Center 2025 | LGPD (Lei 13.709/2018)

---

## 1. WhatsApp Business — Limites e Riscos

### Limites de Envio (outbound frio)

| Tipo de número | Limite diário seguro | Limite máximo estimado |
|---|---|---|
| Número novo (0-7 dias) | 20-30 mensagens | 50 (alto risco) |
| Número em aquecimento (7-30 dias) | 50-80 mensagens | 100 |
| Número aquecido (30+ dias, uso orgânico) | 100-150 mensagens | 200 |
| WhatsApp Business API verificada | Sem limite por número, mas por template aprovado | Escala livre com template |

**Aquecimento de número**: número novo precisa de 7-14 dias de conversas orgânicas (amigos, família, responder mensagens recebidas) antes de outbound em escala. Pular essa etapa = risco de ban imediato.

### Sinais de Alerta de Bloqueio

- **Taxa de bloqueio >10%**: parar todos os envios imediatamente. Revisar lista, script e frequência antes de retomar
- **Taxa de "não entregue" >5%**: número com reputação baixa. Parar e aguardar 48h antes de retomar com volume menor
- **Relatório de spam**: qualquer denúncia formal via WhatsApp é registrada. Após 3 denúncias: ban temporário (7-30 dias)

### Boas Práticas Anti-Bloqueio

1. Nunca enviar link na primeira mensagem
2. Sempre incluir contexto de onde encontrou o lead ("vi seu perfil no Instagram")
3. Personalizar cada mensagem: nome + detalhe do negócio
4. Variar ligeiramente o texto entre envios (evitar mensagens 100% idênticas em série)
5. Respeitar horários comerciais: seg-sex 8h-12h e 14h-18h; sáb 9h-12h
6. Nunca enviar após 19h ou antes das 8h
7. Nunca enviar no domingo
8. Intervalo mínimo de 2-3 minutos entre mensagens enviadas sequencialmente

### Política Anti-Spam do WhatsApp (Meta)

O WhatsApp usa machine learning para detectar padrões de spam. Gatilhos comuns:
- Mesmo texto enviado para múltiplos números em curto intervalo
- Link externo na primeira mensagem
- Alta taxa de não-resposta combinada com bloqueios
- Número novo com volume alto imediatamente

**Consequências**: aviso, limitação de funcionalidades, ban temporário (7-30 dias), ban permanente.

---

## 2. Instagram DM — Limites e Riscos

### Limites de DMs Frios

| Maturidade da conta | DMs frios por dia (seguro) | DMs frios por dia (máximo) |
|---|---|---|
| Conta nova (0-90 dias) | 10-15 | 20 |
| Conta ativa (90+ dias, posts regulares) | 20-30 | 50 |
| Conta verificada ou grande (10k+ seguidores) | 30-50 | 80 |

**Definição de "conta ativa"**: pelo menos 3 posts/semana, histórico de interações reais, não comprar seguidores.

### Regras de Conteúdo no Primeiro Contato

- **Proibido**: links externos, URLs encurtadas, menções a dinheiro ou preço
- **Proibido**: uso da palavra "grátis", "desconto", "promoção" no primeiro toque
- **Obrigatório**: texto que soa pessoal e contextualizado, não de broadcast
- **Recomendado**: mencionar algo específico do perfil do lead (post recente, produto específico)

### Warm-up antes da DM

Antes de enviar DM fria para um lead novo:
1. Curtir 2-3 posts recentes (não todos de uma vez — espaçar 10-15 minutos)
2. Assistir ao story mais recente (se disponível)
3. Opcional: comentário genuíno em um post (aumenta taxa de abertura da DM subsequente)

**Tempo recomendado de warm-up**: 24-48h antes da DM.

### Frequência de Follow-up

- Primeiro toque: dia 1
- Segundo toque no mesmo canal: mínimo 72h após o primeiro
- Terceiro toque no mesmo canal: mínimo 5 dias após o segundo
- Após 3 toques sem resposta no Instagram: migrar para WhatsApp

### Sinais de Conta Restrita (Instagram)

- Ícone de "mensagem solicitacao" que não converte para DM normal
- Conta rotulada como "Conta comercial" pelo Instagram (menor alcance orgânico de DM)
- Mensagens caindo em "Solicitacoes de mensagem" sem notificação para o lead

---

## 3. LGPD — Dados de Leads, Opt-out e Armazenamento

### Base Legal para Outbound Frio (Art. 7, LGPD)

O outbound frio B2B para MEI/pessoa jurídica opera sob:
- **Legítimo interesse** (Art. 7, IX): desde que o interesse comercial seja genuíno e o lead seja informado sobre como seus dados foram obtidos
- **Consentimento implícito**: perfil público no Instagram com número de contato divulgado pelo próprio titular implica aceite de contato comercial

**Limites do legítimo interesse**:
- Dados coletados devem ser usados apenas para o propósito declarado (contato comercial)
- Lead pode solicitar opt-out a qualquer momento
- Nao reter dados após opt-out explícito

### Dados que Podem ser Coletados e Armazenados

Permitido (dados públicos do perfil):
- Nome do negócio
- Cidade/bairro
- Segmento
- Instagram handle
- Número de telefone publicamente exibido
- Número de seguidores
- Observações sobre o negócio (tipo de serviço, horário de funcionamento)

Proibido sem consentimento explícito:
- Dados pessoais do dono (CPF, endereço residencial, dados financeiros)
- Gravações de conversa
- Dados de terceiros mencionados pelo lead

### Opt-out — Protocolo Obrigatório

Se o lead solicitar parar os contatos (qualquer forma: "para de me chamar", "não tenho interesse", "me tire da sua lista"):

1. **Cessar contato imediatamente** pelo canal em que o pedido foi feito
2. **Cessar contato pelos outros canais** (WhatsApp E Instagram) mesmo que o pedido tenha sido só em um
3. **Registrar opt-out** na planilha de leads com data e canal
4. **Não reativar** esse lead por pelo menos 12 meses
5. **Apagar dados** se o lead solicitar explicitamente (direito de exclusão, Art. 18, LGPD)

### Armazenamento Seguro de Leads

- Planilha de leads deve ser armazenada localmente ou em nuvem com acesso restrito
- Nunca compartilhar lista de leads com terceiros sem consentimento dos titulares
- Não armazenar senhas, dados de pagamento ou informações sensíveis
- Período de retenção recomendado: 24 meses para leads que responderam, 6 meses para leads sem resposta
- Após período de retenção: deletar ou anonimizar (substituir nome e telefone por ID genérico)

### Identificação no Contato

Na primeira mensagem, deve ser possível identificar quem está entrando em contato. Não é obrigatorio escrever CNPJ, mas o nome do negócio/solução deve aparecer naturalmente na conversa antes de qualquer proposta comercial.
