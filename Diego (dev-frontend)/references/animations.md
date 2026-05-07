# Animações — Framer Motion v11 + View Transitions API

## Quando usar cada um

| Cenário | Ferramenta |
|---------|-----------|
| Transição entre rotas (simples crossfade/slide) | View Transitions API (nativo, zero JS overhead) |
| Transições de página com física spring | Framer Motion `AnimatePresence` em `template.tsx` |
| Lista com itens add/remove (reflow suave) | `motion` + `layout` + `AnimatePresence` |
| Card → modal expand (shared element) | Framer Motion `layoutId` |
| Ambos no mesmo app | Sim — coexistem (VTA nas rotas, FM nos componentes) |
| Data tables e dashboards densos | Nenhum — performance > estética |

**Regra:** Não animar o mesmo elemento com VTA e `AnimatePresence` simultaneamente — browser VTA e FM JS-driven exit vão competir.

---

## View Transitions API — Next.js 15

### Configuração

```ts
// next.config.ts
import type { NextConfig } from 'next'
const nextConfig: NextConfig = {
  experimental: { viewTransition: true },
}
export default nextConfig
```

### CSS personalizado

```css
/* app/globals.css */
::view-transition-old(page-content) { animation: 200ms ease-out fade-out; }
::view-transition-new(page-content) { animation: 200ms ease-in fade-in; }

@keyframes fade-out { from { opacity: 1 } to { opacity: 0 } }
@keyframes fade-in  { from { opacity: 0 } to { opacity: 1 } }
```

```tsx
// view-transition-name deve ser único por elemento visível
<div style={{ viewTransitionName: 'page-content' }}>
  {/* conteúdo */}
</div>
```

### Link programático com VTA

```tsx
'use client'
import { useRouter } from 'next/navigation'

export function TransitionLink({ href, children }: { href: string; children: React.ReactNode }) {
  const router = useRouter()
  return (
    <a
      href={href}
      onClick={(e) => {
        e.preventDefault()
        if (!document.startViewTransition) { router.push(href); return }
        document.startViewTransition(() => { router.push(href) })
      }}
    >
      {children}
    </a>
  )
}
```

---

## Framer Motion v11 — Padrões para App Router

### GOTCHA CRÍTICO: `template.tsx` não `layout.tsx`

`layout.tsx` persiste entre navegações → componente **não desmonta** → `AnimatePresence` nunca roda a animação de exit.

`template.tsx` remonta a cada navegação → desmount/mount correto → `AnimatePresence` funciona.

```tsx
// app/(app)/template.tsx  ← CORRETO
// app/(app)/layout.tsx    ← ERRADO para AnimatePresence
'use client'
import { motion, AnimatePresence } from 'framer-motion'
import { usePathname } from 'next/navigation'

const pageVariants = {
  initial: { opacity: 0, y: 8 },
  enter:   { opacity: 1, y: 0, transition: { duration: 0.2, ease: 'easeOut' } },
  exit:    { opacity: 0, y: -8, transition: { duration: 0.15, ease: 'easeIn' } },
}

export default function Template({ children }: { children: React.ReactNode }) {
  const pathname = usePathname()
  return (
    <AnimatePresence mode="wait">
      <motion.div key={pathname} variants={pageVariants} initial="initial" animate="enter" exit="exit">
        {children}
      </motion.div>
    </AnimatePresence>
  )
}
```

### Layout Animations — lista com add/remove

`layout` prop = FLIP animations (repositionamento suave de irmãos).
`AnimatePresence` = mount/unmount com animação.

```tsx
'use client'
import { motion, AnimatePresence } from 'framer-motion'

export function AnimatedLeadList({ leads }: { leads: Lead[] }) {
  return (
    <ul className="space-y-2">
      <AnimatePresence initial={false}>
        {leads.map((lead) => (
          <motion.li
            key={lead.id}
            layout                              // FLIP: irmãos reposicionam suavemente
            initial={{ opacity: 0, x: -16 }}
            animate={{ opacity: 1, x: 0 }}
            exit={{ opacity: 0, x: 16, transition: { duration: 0.15 } }}
            transition={{ type: 'spring', stiffness: 300, damping: 28 }}
            className="rounded-md border p-3"
          >
            {lead.nome}
          </motion.li>
        ))}
      </AnimatePresence>
    </ul>
  )
}
```

### Shared Element Transition — card para modal (`layoutId`)

```tsx
'use client'
import { useState } from 'react'
import { motion, AnimatePresence } from 'framer-motion'

export function LeadCardGrid({ leads }: { leads: Lead[] }) {
  const [selected, setSelected] = useState<Lead | null>(null)

  return (
    <>
      <div className="grid grid-cols-3 gap-4">
        {leads.map((lead) => (
          <motion.div
            key={lead.id}
            layoutId={`lead-card-${lead.id}`}  // ponte para o modal
            onClick={() => setSelected(lead)}
            className="cursor-pointer rounded-lg bg-muted p-4"
          >
            <motion.h3 layoutId={`lead-name-${lead.id}`}>{lead.nome}</motion.h3>
          </motion.div>
        ))}
      </div>

      <AnimatePresence>
        {selected && (
          <>
            {/* Overlay */}
            <motion.div
              initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}
              className="fixed inset-0 z-40 bg-black/50"
              onClick={() => setSelected(null)}
            />
            {/* Modal com mesmo layoutId */}
            <motion.div
              layoutId={`lead-card-${selected.id}`}
              className="fixed inset-10 z-50 rounded-xl bg-background p-8 shadow-2xl"
            >
              <motion.h2 layoutId={`lead-name-${selected.id}`} className="text-2xl font-bold">
                {selected.nome}
              </motion.h2>
              <button onClick={() => setSelected(null)}>Fechar</button>
            </motion.div>
          </>
        )}
      </AnimatePresence>
    </>
  )
}
```

### Micro-interações utilitárias

```tsx
// Botão com feedback tátil
<motion.button
  whileHover={{ scale: 1.02 }}
  whileTap={{ scale: 0.98 }}
  transition={{ type: 'spring', stiffness: 400, damping: 17 }}
>
  Salvar
</motion.button>

// Skeleton → conteúdo (fade in suave)
<AnimatePresence mode="wait">
  {isPending
    ? <motion.div key="skeleton" exit={{ opacity: 0 }}><Skeleton /></motion.div>
    : <motion.div key="content" initial={{ opacity: 0 }} animate={{ opacity: 1 }}>{content}</motion.div>
  }
</AnimatePresence>
```

### `prefers-reduced-motion`

```tsx
// Respeitar preferências de acessibilidade
import { useReducedMotion } from 'framer-motion'

export function AnimatedCard({ children }: { children: React.ReactNode }) {
  const prefersReduced = useReducedMotion()
  return (
    <motion.div
      initial={{ opacity: 0, y: prefersReduced ? 0 : 12 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: prefersReduced ? 0 : 0.2 }}
    >
      {children}
    </motion.div>
  )
}
```

---

## Fontes
- https://www.framer.com/motion/animate-presence/
- https://www.framer.com/motion/layout-animations/
- https://nextjs.org/docs/app/api-reference/config/next-config-js/viewTransition
- https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API
