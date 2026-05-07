# SOPs — Procedimentos Operacionais Padrão

> Checklists e protocolos reproduzíveis para cada etapa do workflow de desenvolvimento de LPs.

---

## SOP 1 — Intake de Novo Cliente

### Objetivo
Garantir que toda informação necessária para desenvolver a LP seja coletada antes de iniciar qualquer trabalho.

### Checklist de Campos Obrigatórios do Brief

**Negócio:**
- [ ] Nome do negócio / marca
- [ ] Segmento (barbearia, personal trainer, coach, infoprodutor, etc.)
- [ ] Cidade/região de atuação (crítico para social proof regional)
- [ ] Objetivo da LP: lead / cadastro / venda / agendamento
- [ ] URL atual do negócio (se existir) ou domínio desejado

**Público e Oferta:**
- [ ] Público-alvo: perfil, dores principais, desejos
- [ ] Produto/serviço principal sendo ofertado
- [ ] Diferencial competitivo (o que te diferencia da concorrência)
- [ ] Preço ou faixa de preço (se aparecer na LP)
- [ ] Urgência ou escassez real (se houver)

**Copy (a ser finalizado pelo copywriter):**
- [ ] Headline candidata (cliente fornece rascunho)
- [ ] CTAs desejados: texto + destino (WhatsApp, formulário, checkout)
- [ ] Tom de voz: formal / descontraído / urgente / premium
- [ ] Objeções mais comuns dos clientes (para o FAQ)

**Visual:**
- [ ] Logo em SVG ou PNG com fundo transparente
- [ ] Paleta de cores (hex ou referência)
- [ ] Fontes preferidas (ou referência de site que gosta)
- [ ] Fotos do negócio (obrigatório para presenciais)
- [ ] Vídeo de apresentação (se disponível)

**Técnico:**
- [ ] Integração necessária: CRM / e-mail marketing / WhatsApp Business / checkout
- [ ] Domínio: cliente tem? Quem gerencia o DNS?
- [ ] Google Analytics existente? (ID da propriedade)

**Prazo e Entrega:**
- [ ] Data de go-live definida
- [ ] Responsável pelo feedback do cliente (nome + contato direto)
- [ ] Aprovação formal: como será feita (e-mail, WhatsApp)

### SLA de Resposta

| Etapa | Prazo |
|-------|-------|
| Confirmação de recebimento do brief | 24h |
| Devolutiva com dúvidas ou pedido de completar campos | 48h |
| Kickoff agendado | 72h após brief completo |

---

## SOP 2 — Setup de Novo Projeto

### Objetivo
Configurar repositório, infraestrutura e ferramentas de monitoring antes de escrever a primeira linha de código da LP.

### Checklist de Setup

**1. Repositório Git**
```bash
# Criar e inicializar
mkdir cliente-nome-lp && cd cliente-nome-lp
git init
git remote add origin https://github.com/usuario/cliente-nome-lp

# Criar .gitignore com Next.js defaults
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir
```

**2. Estrutura inicial**
- [ ] `src/types/database.types.ts` — placeholder (gerar após criar tabelas)
- [ ] `themes/cliente-nome.ts` — tokens do Figma
- [ ] `src/schemas/forms.ts` — schemas Zod para formulários
- [ ] `.env.local` com variáveis de ambiente (NUNCA commitar)
- [ ] `.env.example` com chaves sem valores (commitar para referência)

**3. Vercel**
```bash
vercel link  # vincular ao projeto existente ou criar novo
# Ou via Vercel MCP após instalar
```
- [ ] Environment variables configuradas no dashboard Vercel
- [ ] Framework preset: Next.js detectado automaticamente

**4. Supabase (via MCP)**
```
mcp__claude_ai_Supabase__create_project ou selecionar projeto existente
mcp__claude_ai_Supabase__apply_migration → criar tabela leads mínima
mcp__claude_ai_Supabase__generate_typescript_types → salvar em src/types/
```

Tabela de leads mínima:
```sql
CREATE TABLE leads (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  nome TEXT NOT NULL,
  email TEXT,
  whatsapp TEXT,
  origem TEXT DEFAULT 'lp-principal',
  criado_em TIMESTAMPTZ DEFAULT NOW()
);

-- RLS obrigatório
ALTER TABLE leads ENABLE ROW LEVEL SECURITY;

-- Somente service_role pode ler (segurança)
CREATE POLICY "Service role only"
  ON leads FOR ALL
  USING (false)
  WITH CHECK (false);
```

**5. Sentry**
```bash
npx @sentry/wizard@latest -i nextjs
```
- [ ] DSN configurado no Vercel (environment variable)
- [ ] Integração Vercel ↔ Sentry ativada no marketplace (source maps automáticos)
- [ ] Testar: lançar erro manual e verificar no dashboard Sentry

**6. Monitoramento**
```bash
npm install @vercel/speed-insights @vercel/analytics
```

Adicionar em `app/layout.tsx`:
```tsx
import { SpeedInsights } from '@vercel/speed-insights/next'
import { Analytics } from '@vercel/analytics/react'

// No return do RootLayout:
<SpeedInsights />
<Analytics />
```

**7. TypeScript config**
```bash
# Verificar que tsc passa sem erros antes do primeiro commit
npx tsc --noEmit
```

**8. Primeiro commit**
```bash
git add .
git commit -m "chore: setup inicial — Next.js + Supabase + Sentry + Vercel"
git push -u origin main
```

### Variáveis de Ambiente Obrigatórias

```bash
# .env.local (não commitar)
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...  # somente server-side
SENTRY_DSN=https://xxx@sentry.io/xxx
SENTRY_AUTH_TOKEN=xxx
SENTRY_ORG=triforce-auto
SENTRY_PROJECT=cliente-nome
```

---

## SOP 3 — Go-Live (10 Itens)

### Objetivo
Garantir que a LP vai ao ar corretamente, sem problemas técnicos que impactem a experiência do usuário ou as métricas de conversão.

### Checklist de Go-Live

**DNS e Segurança:**
1. [ ] **DNS propagado** — registro A ou CNAME do domínio apontando para Vercel. Verificar com `dig nome-dominio.com` ou [dnschecker.org](https://dnschecker.org). Aguardar propagação completa (pode levar até 24h, geralmente < 1h).
2. [ ] **SSL ativo** — certificado TLS emitido automaticamente pela Vercel via Let's Encrypt. Verificar HTTPS sem aviso de segurança no browser.

**CDN e Performance:**
3. [ ] **CDN configurado** — Vercel Edge Network está ativo por padrão. Verificar headers `cf-cache-status` ou `x-vercel-cache` na resposta (deve ser HIT em assets estáticos).
4. [ ] **Redirects 301 configurados** — se há URLs antigas (de site anterior), adicionar redirects em `next.config.ts` para não perder tráfego/SEO.
5. [ ] **Build de produção limpo** — zero erros TypeScript, zero warnings de lint críticos. Verificar no dashboard Vercel que o build passou sem erros.

**SEO:**
6. [ ] **Sitemap.xml** acessível em `https://dominio.com/sitemap.xml`. Implementar via `app/sitemap.ts` do Next.js.
7. [ ] **robots.txt** configurado em `app/robots.ts`. Verificar que não está bloqueando Googlebot.

**Funcionalidade:**
8. [ ] **Formulário de lead testado** — submeter formulário real com dados de teste → verificar confirmação → verificar dado no Supabase → verificar que e-mail de notificação chegou (se configurado).

**Analytics e Monitoramento:**
9. [ ] **Analytics ativos** — Vercel Analytics e Speed Insights coletando primeiros dados. Se Google Analytics for necessário, verificar ID configurado e evento `page_view` chegando.
10. [ ] **Sentry + alertas configurados** — Sentry recebendo eventos. Vercel Error Anomaly Alert ativado (dashboard Vercel → projeto → Monitoring → Alerts).

### Comunicação ao Cliente

Após go-live confirmar:
- URL de produção funcionando
- Formulário testado por você antes de passar para o cliente
- Instruções básicas: como ver leads no Supabase, como acompanhar analytics
- Aviso sobre pausa do Supabase (se free tier): "O banco pode pausar após 7 dias sem visitas. Me avise se isso acontecer."

---

## SOP 4 — Rotina Semanal de Monitoramento

### Objetivo
Identificar problemas antes que o cliente perceba e manter qualidade técnica das LPs em produção.

### Checklist Semanal (estimativa: 20–30 minutos por LP ativa)

**Vercel Dashboard:**
- [ ] Verificar se há builds com erro nos últimos 7 dias
- [ ] Checar Runtime Logs (a janela de 1h do Free tier) para erros recorrentes
- [ ] Verificar se algum deploy falhou e foi revertido automaticamente

**Vercel Speed Insights:**
- [ ] LCP ainda abaixo de 2.5s? (P75)
- [ ] INP ainda abaixo de 200ms? (P75)
- [ ] CLS ainda abaixo de 0.1? (P75)
- [ ] Se alguma métrica piorou: identificar deployment causador e investigar

**Sentry:**
- [ ] Novos errors na última semana?
- [ ] Error rate subiu sem motivo aparente?
- [ ] Resolver ou marcar como ignorado erros conhecidos para não poluir o dashboard

**Supabase (Free tier):**
- [ ] Projeto ainda ativo (não pausado)?
- [ ] Se inativo há mais de 5 dias: contatar cliente para verificar se tem tráfego ou fazer ping manual

**Vercel Analytics:**
- [ ] Volume de visitas dentro do esperado?
- [ ] Alguma queda abrupta de tráfego (pode indicar problema de DNS ou indexação)?

---

## SOP 5 — Entrega ao Cliente

### Objetivo
Garantir transição suave do ambiente de staging para produção com aprovação formal do cliente.

### Etapas

**1. Staging (Preview URL)**
- Enviar URL de preview do Vercel ao cliente
- Solicitar revisão em todos os dispositivos (mobile obrigatório)
- Aguardar feedback por escrito (e-mail ou checklist compartilhado)

**2. Rodada de Ajustes**
- Máximo 2 rodadas de ajustes incluídas no escopo padrão
- Ajustes documentados em issue do GitHub ou lista de tarefas
- Cada rodada de ajustes: 48–72h para implementar e enviar nova preview

**3. Aprovação Formal**
- Cliente confirma por e-mail: "Aprovado para ir ao ar" (ou equivalente)
- Guardar este e-mail — é a documentação de aceite da entrega

**4. Go-live**
- Executar SOP 3 — Go-Live (10 Itens)
- Deploy para produção: merge da branch aprovada para `main` → Vercel deploy automático

**5. Handoff Técnico**
- Documento de handoff (pode ser e-mail ou Notion) com:
  - URL de produção
  - Credenciais de acesso ao painel Supabase (somente leitura)
  - Link para Vercel Analytics
  - Instruções para acessar leads no Supabase Table Editor
  - Contato de suporte (Triforce Auto) para problemas técnicos

**6. Atualizar `accounts.yaml`**
- Marcar LP como `status: live`
- Registrar data de go-live
- Registrar URL de produção, project IDs (Vercel, Supabase, Sentry)

### Template de E-mail de Entrega

```
Assunto: [NOME DO NEGÓCIO] — LP no ar ✓

[Nome do cliente],

Sua landing page está no ar!

URL de produção: https://seudominio.com.br
Painel de leads: [link direto para Supabase Table Editor]
Analytics: [link para Vercel Analytics]

O que está monitorado automaticamente:
- Erros técnicos (Sentry)
- Velocidade da página (Vercel Speed Insights)
- Visitas e pageviews (Vercel Analytics)

Próximos passos:
- Compartilhe a URL nas suas redes
- Em 30 dias, faço uma revisão das métricas e te envio um relatório

Qualquer dúvida técnica, é só me chamar.

[Assinatura]
```
