# Estratégico — Frameworks e Padrões

> Fonte: research/dev-web/scan-estrategico.md + validacao-estrategica.md (2026-04-13)

---

## 1. LP Workflow — 8 Fases Completo

### Fase 1 — Intake de Brief

Formulário padronizado com campos obrigatórios:

| Campo | Descrição |
|-------|-----------|
| Objetivo da LP | Lead / cadastro / venda direta |
| Público-alvo | Persona principal (idade, dores, desejos) |
| Diferenciais | O que diferencia o negócio da concorrência |
| Headlines candidatas | Ideias do cliente (copywriter finaliza) |
| CTAs desejados | Texto dos botões + destino (WhatsApp, formulário, checkout) |
| Tom de voz | Formal / descontraído / urgente / premium |
| Seções desejadas | Quais das 9 seções são obrigatórias |
| Recursos visuais | Logo (SVG/PNG), paleta de cores, fontes preferidas |
| Foto/vídeo | Material visual do negócio (obrigatório para LPs presenciais) |
| Requisitos técnicos | Integração com CRM, e-mail, WhatsApp Business, checkout |
| KPIs | Meta de conversão, volume de leads esperado |
| Prazo | Data de go-live definida |
| Orçamento | Confirmar faixa contratada |

Saída: brief assinado via e-mail + one-pager resumido para referência rápida durante o dev.

### Fase 2 — Aprovação de Design (Sprint 5 Dias)

```
Day 0: Kickoff — brief resumido + agenda
Day 1–2: Wireframes baixa fidelidade (hierarquia, CTA placement, fluxo mobile)
Day 3–4: Mockups alta fidelidade (cores, tipografia, estados de hover/focus)
Day 5: Link Figma para aprovação (protótipo navegável opcional)
```

Critérios de aprovação mínimos:
- Estrutura de seções aprovada
- Paleta de cores e tipografia aprovadas
- Responsividade em 3 breakpoints (desktop 1280px, tablet 768px, mobile 375px)

Entregável: link Figma + design tokens exportados (cores, tipografia, spacing, radii).

### Fase 3 — Preparação Técnica

Checklist de setup:

```bash
# 1. Repositório
git init
git remote add origin https://github.com/usuario/cliente-lp

# 2. Next.js + TypeScript
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir

# 3. Dependências essenciais
npm install @supabase/supabase-js @supabase/ssr
npm install @sentry/nextjs
npm install @vercel/speed-insights @vercel/analytics

# 4. Supabase (via MCP ou CLI)
# Criar projeto → gerar tipos → copiar para src/types/database.types.ts

# 5. Vercel
vercel link  # ou via MCP após instalar Vercel MCP

# 6. Design tokens
# Criar themes/cliente-nome.ts com tokens do Figma
```

### Fase 4 — Desenvolvimento

Estrutura de diretórios padrão:

```
src/
  app/
    layout.tsx      # RootLayout com SpeedInsights + Analytics
    page.tsx        # Landing page principal
    api/            # Route Handlers (formulários, webhooks)
  components/
    ui/             # Atoms reutilizáveis (Button, Input, Badge)
    sections/       # Seções da LP (Hero, Testimonials, FAQ, etc)
    layout/         # Header, Footer (se aplicável)
  lib/
    supabase.ts     # Cliente Supabase tipado singleton
    utils.ts        # Helpers
  types/
    database.types.ts   # Gerado via Supabase CLI/MCP
    index.ts            # Re-exports e tipos customizados
  schemas/
    forms.ts        # Schemas Zod para validação de formulários
themes/
  cliente-nome.ts   # Design tokens do cliente
```

Git branching simples:
- `main` — produção
- `feature/cliente-secao` — desenvolvimento por seção/feature
- `hotfix/descricao` — correções urgentes em produção

### Fase 5 — QA Checklist

- [ ] Responsividade: testar em 375px, 768px, 1280px
- [ ] CTAs e formulários funcionando com integração real
- [ ] Lighthouse score ≥ 90 (Performance + Acessibilidade + SEO)
- [ ] LCP < 2.5s no modo mobile simulado
- [ ] Sem placeholders visíveis (Lorem ipsum, imagens de stock genéricas)
- [ ] Links ativos (sem 404)
- [ ] Alt text em todas as imagens
- [ ] Formulário de lead: dados chegando no Supabase

### Fase 6 — Entrega ao Cliente

- Enviar URL de preview com formulário de aprovação (pode ser e-mail simples)
- Aguardar aprovação formal antes de ir para produção
- Documentação técnica básica: tokens usados, APIs integradas, como atualizar conteúdo

### Fase 7 — Go-Live Checklist (10 Itens)

1. [ ] DNS: registro A/CNAME propagado (verificar com `dig` ou DNS Checker)
2. [ ] SSL: certificado ativo e renovação automática configurada
3. [ ] CDN: Vercel Edge Network ativo (padrão), headers de cache configurados
4. [ ] Redirects 301: URLs antigas redirecionando corretamente
5. [ ] Sitemap.xml acessível em `/sitemap.xml`
6. [ ] robots.txt configurado corretamente
7. [ ] Build de produção limpo (zero warnings de TypeScript, zero erros de lint)
8. [ ] Formulário de lead testado em produção (submissão real → confirmação → dado no Supabase)
9. [ ] Vercel Analytics + Speed Insights coletando dados
10. [ ] Sentry recebendo eventos (testar com erro manual → verificar no dashboard)

### Fase 8 — Pós-Lançamento

- Verificação semanal de Vercel Speed Insights e Sentry
- Relatório mensal ao cliente (visitas, leads captados, CWV)
- Roadmap de melhorias (priorizado por impacto em conversão)

---

## 2. Sistema de Design Tokens-First

### Filosofia

Nunca hardcodar valores visuais em componentes. Todo projeto começa com um arquivo de tokens que define a identidade visual do cliente. Isso permite:
- Reutilizar 100% dos componentes entre clientes
- Mudar toda a identidade visual alterando apenas um arquivo
- Onboarding do 2º, 3º e 4º cliente em horas, não dias

### Estrutura de Tokens

```typescript
// themes/base.ts — tokens padrão (fallback)
export const baseTokens = {
  colors: {
    primary: '#000000',
    accent: '#000000',
    bg: '#ffffff',
    text: '#111111',
    muted: '#6b7280',
    surface: '#f9fafb',
    border: '#e5e7eb',
  },
  radii: {
    sm: '6px',
    md: '12px',
    lg: '24px',
    full: '9999px',
  },
  shadows: {
    sm: '0 1px 2px rgba(0,0,0,.05)',
    md: '0 4px 16px rgba(0,0,0,.1)',
    lg: '0 8px 32px rgba(0,0,0,.15)',
  },
  spacing: {
    xs: '4px', sm: '8px', md: '16px',
    lg: '32px', xl: '64px', '2xl': '96px',
  },
  breakpoints: {
    sm: '640px', md: '768px', lg: '1024px', xl: '1280px',
  },
  fontSizes: {
    sm: '0.875rem', base: '1rem', lg: '1.125rem',
    xl: '1.25rem', '2xl': '1.5rem', '3xl': '1.875rem', '4xl': '2.25rem',
  },
}

// themes/barbearia-silva.ts — tema por cliente
import { baseTokens } from './base'
export const barbeariaVintage = {
  ...baseTokens,
  colors: {
    ...baseTokens.colors,
    primary: '#c8860a',   // dourado
    accent: '#8b1a1a',    // vinho
    bg: '#1a1a1a',        // fundo escuro
    text: '#f5f0e8',      // creme
    surface: '#2a2a2a',
    border: '#3a3a3a',
  },
}

// themes/coach-premium.ts
export const coachPremium = {
  ...baseTokens,
  colors: {
    ...baseTokens.colors,
    primary: '#2563eb',
    accent: '#7c3aed',
    bg: '#ffffff',
    text: '#0f172a',
  },
}
```

### Integração com Tailwind CSS

```typescript
// tailwind.config.ts
import { clienteTema } from './themes/cliente-nome'

export default {
  theme: {
    extend: {
      colors: {
        primary: clienteTema.colors.primary,
        accent: clienteTema.colors.accent,
        // ...
      },
    },
  },
}
```

### Fluxo de Entrega por Cliente (2º em diante)

1. Selecionar template base (`LPTemplateBasic` ou `LPTemplateFullSales`)
2. Criar `themes/cliente-nome.ts` com tokens do Figma
3. Criar `content/cliente-nome.json` com textos (fornecidos pelo copywriter)
4. Aplicar `<ThemeProvider theme={clienteTema}>` no root layout
5. Build e deploy

---

## 3. LP Conversion Patterns

### Estrutura das 9 Seções — Detalhes de Implementação

#### Seção 1 — Hero

Obrigações técnicas:
- CTA visível **sem scroll** em todos os dispositivos (acima da dobra)
- Texto antes da imagem no DOM (mobile-first rendering)
- Imagem hero: formato WebP, dimensões explícitas (evita CLS), `loading="eager"` (é LCP)
- Fonte do headline: carregada via `next/font` com `display: swap`

```tsx
// Estrutura do Hero component
<section>
  <div> {/* conteúdo textual */}
    <h1>{headline}</h1>      {/* headline em 5–8 palavras */}
    <p>{subheadline}</p>     {/* prova social + localização */}
    <CTAButton />            {/* obrigatório acima da dobra */}
    <SocialProof />          {/* ★★★★★ avaliações Google */}
  </div>
  <div> {/* imagem */}
    <Image
      src={heroImage}
      alt={altText}
      width={800} height={600}  {/* dimensões explícitas — evita CLS */}
      priority                   {/* LCP element — carrega imediatamente */}
    />
  </div>
</section>
```

#### Seção 2 — Prova Social (Depoimentos)

- Foto + nome + **cidade** — social proof regional aumenta conversão em nichos locais
- Mínimo 3 depoimentos, máximo 6 na tela inicial
- Avaliações reais do Google/Facebook quando disponíveis
- Formato: card simples com foto, citação, nome, cidade e estrelas

#### Seção 3 — Problema/Dor

- Identificação com a dor do cliente antes de apresentar a solução
- Tom: empático, não alarmista
- 2–3 parágrafos curtos ou bullets

#### Seção 4 — Solução/Benefícios

- 3–5 benefícios tangíveis (não features)
- Cada benefício: ícone + título + 1 linha de descrição
- Linguagem do resultado: "Você vai conseguir X" não "Nossa solução faz Y"

#### Seção 5 — Como Funciona

- Processo em 3 passos simples
- Números grandes e visíveis
- Simplificar ao máximo — cliente não quer complexidade

#### Seção 6 — Prova Adicional

- Cases de sucesso com resultados específicos
- Antes/depois quando aplicável
- Logos de parceiros ou clientes (se disponível)

#### Seção 7 — Oferta/Preço

- Pricing claro — sem "entre em contato para saber o preço"
- Urgência real (não falsa): vagas limitadas, prazo de oferta real
- CTA de conversão direto: "Garantir minha vaga agora"

#### Seção 8 — FAQ

- 5–8 perguntas respondendo as principais objeções
- Accordion para não poluir a página
- JSON-LD FAQPage schema para SEO

#### Seção 9 — CTA Final

- Reforço da oferta principal
- Último CTA da página
- Urgência reforçada se aplicável

### Posicionamento de CTAs

| Posição | Função | Cor |
|---------|--------|-----|
| Hero (acima da dobra) | Principal — primeiro contato | Alta contraste com bg |
| Após seção 2 (prova social) | Reforço pós-credibilidade | Mesmo estilo do principal |
| Após pricing (seção 7) | Conversão direta | Destaque máximo |
| Footer / CTA Final | Última chance | Mesmo do principal |
| Mobile sticky bottom | Sempre visível durante scroll | Compacto, ícone + texto |

Regra: mínimo 2 CTAs, máximo 4. Textos de ação: "Agendar agora" > "Clique aqui".

### Mobile-First — Regras Críticas

- Botão mínimo: **44px de altura** (touch target iOS/Android)
- Tipografia mínima: **16px para corpo** (evita zoom automático iOS)
- Formulário: máximo **3–4 campos** no mobile
- Sticky bottom button: sempre visível, não sobrepor conteúdo importante
- Teste obrigatório em **375px de largura** (iPhone SE — menor tela comum)

---

## 4. Core Web Vitals — Diagnóstico e Correção

### O que cada métrica mede

**LCP (Largest Contentful Paint)** — quanto tempo até o maior elemento visível carregar. Geralmente é a imagem hero ou o headline principal. Medir no P75.

**INP (Interaction to Next Paint)** — tempo de resposta a interações do usuário (cliques, teclado, touch). Substituiu o FID em 2024. Medir no P75.

**CLS (Cumulative Layout Shift)** — quanto o layout muda inesperadamente durante o carregamento. Causado por imagens sem dimensões, fontes carregando com fallback, ads inseridos dinamicamente.

### Como Diagnosticar

Ferramentas em ordem de prioridade:
1. **Vercel Speed Insights** — dados reais de usuários (RUM), P75, por deployment
2. **PageSpeed Insights (PSI)** — lab data + field data (CrUX)
3. **Chrome DevTools → Performance → Web Vitals** — debug local detalhado
4. **Lighthouse** — auditoria completa (performance, acessibilidade, SEO, boas práticas)

### Como Corrigir

**LCP alto (> 2.5s):**
- Imagem hero sem `priority` no Next.js Image → adicionar `priority`
- Imagem hero em formato JPEG grande → converter para WebP e usar `next/image`
- Fonte do headline bloqueando renderização → usar `next/font` com `display: 'swap'`
- TTFB alto → verificar região do servidor Vercel, configurar ISR/cache

**INP alto (> 200ms):**
- JavaScript pesado no main thread → code splitting com `dynamic()` do Next.js
- Event handlers síncronos pesados → mover processamento para Web Workers ou Server Actions
- Hydration pesada → verificar se componente realmente precisa ser Client Component

**CLS alto (> 0.1):**
- Imagens sem `width` e `height` → sempre especificar dimensões no `<Image />`
- Fontes causando FOUT/FOIT → usar `next/font` (carrega fonte com display: swap automaticamente)
- Conteúdo inserido dinamicamente acima do fold → reservar espaço com `min-height`
- Ads ou embeds sem dimensões fixas → container com aspect-ratio fixo

---

## 5. Edge vs Serverless — Tabela Decisória Completa

### Benchmarks Reais (Validados — OpenStatus, 2024)

| Métrica | Edge (V8 Isolate) | Node.js Serverless |
|---------|------------------|--------------------|
| Cold start P50 | ~106ms | ~859ms |
| Warm execution P50 | ~106ms | ~246ms |
| Vantagem cold start | — | 9x mais lento |

> **Aviso Vercel 2025/2026:** A Vercel recomenda **migrar Edge → Node.js** para melhor performance e confiabilidade com Fluid Compute. O gap de cold start diminuiu significativamente.

### Tabela de Decisão

| Use Edge Runtime | Use Node.js (padrão) |
|-----------------|----------------------|
| `middleware.ts` — auth check | Route Handlers com lógica de negócio |
| Redirects e rewrites condicionais | Queries Supabase diretas |
| A/B testing via cookie | Processamento de formulários |
| Geolocalização (usar `@vercel/functions`) | Upload de arquivos |
| Header manipulation (CSP, CORS) | Envio de e-mails |
| i18n routing | Integração com APIs externas pesadas |
| Respostas leves de < 1 MB bundle | Qualquer pacote com APIs Node.js nativas |

### Padrão de Geolocalização (Next.js 15)

```typescript
// middleware.ts — padrão correto para Next.js 15
import { NextRequest, NextResponse } from 'next/server'
import { geolocation } from '@vercel/functions'  // NÃO usar req.geo (removido!)

export function middleware(req: NextRequest) {
  const { city, country, region } = geolocation(req)

  // Exemplo: personalizar CTA por região
  const response = NextResponse.next()
  response.headers.set('x-user-country', country ?? 'BR')
  return response
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
}
```

---

## 6. TypeScript Avançado — Padrão Supabase

### Geração de Tipos

```bash
# Remoto (produção)
npx supabase gen types typescript --project-id "$PROJECT_REF" --schema public > src/types/database.types.ts

# Local (dev com Docker)
npx supabase gen types typescript --local > src/types/database.types.ts

# Via MCP (sem CLI) — mais simples para setup inicial
# mcp__claude_ai_Supabase__generate_typescript_types({ project_id: "xxx" })
# Copiar output para src/types/database.types.ts
```

### Cliente Tipado Singleton

```typescript
// lib/supabase/client.ts (Client Components)
import { createBrowserClient } from '@supabase/ssr'
import type { Database } from '@/types/database.types'

export function createClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}

// lib/supabase/server.ts (Server Components e Route Handlers)
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'
import type { Database } from '@/types/database.types'

export async function createClient() {
  const cookieStore = await cookies()  // await obrigatório no Next.js 15!
  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    { cookies: { getAll: () => cookieStore.getAll() } }
  )
}
```

### Queries Tipadas com QueryData

```typescript
import type { QueryData } from '@supabase/supabase-js'

// Tipo inferido automaticamente para queries simples
const { data: leads } = await supabase.from('leads').select('*')
// data: Database['public']['Tables']['leads']['Row'][] | null

// Para joins complexos: usar QueryData
const query = supabase
  .from('leads')
  .select('id, nome, email, campanha_id(id, nome, cliente_id(nome))')

type LeadsComCampanha = QueryData<typeof query>

const { data, error } = await query
const leads: LeadsComCampanha = data ?? []
```

### Validação com Zod em Route Handlers

```typescript
// app/api/leads/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'
import { createClient } from '@/lib/supabase/server'

const LeadSchema = z.object({
  nome: z.string().min(2).max(100),
  email: z.string().email(),
  whatsapp: z.string().regex(/^\d{10,11}$/),
  origem: z.string().optional(),
})

export async function POST(req: NextRequest) {
  const body = await req.json()
  const result = LeadSchema.safeParse(body)

  if (!result.success) {
    return NextResponse.json(
      { error: result.error.flatten() },
      { status: 400 }
    )
  }

  const supabase = await createClient()
  const { data, error } = await supabase
    .from('leads')
    .insert(result.data)
    .select()
    .single()

  if (error) return NextResponse.json({ error }, { status: 500 })
  return NextResponse.json(data, { status: 201 })
}
```

### Checklist de TypeScript por Projeto

- [ ] `database.types.ts` gerado e versionado no repositório
- [ ] Clientes Supabase tipados com `Database` generic
- [ ] Schemas Zod criados para todos os formulários/API routes
- [ ] `QueryData<typeof query>` para todas as queries com joins
- [ ] `tsc --noEmit` passando sem erros antes de cada deploy
- [ ] Regenerar tipos após cada migration aplicada
