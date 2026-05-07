# Formulários + RBAC — React Hook Form v7 + Zod + shadcn/ui + Supabase Auth

## React Hook Form v7 + Zod + shadcn/ui

### Anatomia da composição shadcn/ui

```
<Form>           → spread do objeto `form` do RHF (provê FormProvider context)
  <FormField>    → conecta ao RHF via `control` + `name`
    <FormItem>   → wrapper de layout
      <FormLabel>   → lê label do contexto do FormField
      <FormControl> → wraps o input, injeta aria-* e id
      <FormMessage> → lê fieldState.error.message automaticamente (sem wiring manual)
```

### Setup completo

```ts
// lib/schemas/leads.ts — schema Zod derivado de drizzle-zod
import { createInsertSchema } from 'drizzle-zod'
import { leads } from '@/db/schema/crm'
import { z } from 'zod'

export const createLeadSchema = createInsertSchema(leads, {
  nome: (s) => s.min(1, 'Nome obrigatório'),
  telefone: z.string().regex(/^\+?[\d\s-]{8,}$/, 'Telefone inválido'),
  email: (s) => s.email('Email inválido').optional(),
}).omit({ id: true, createdAt: true, createdBy: true })

export type CreateLeadValues = z.infer<typeof createLeadSchema>
```

```tsx
// components/leads/create-lead-form.tsx
'use client'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { useAction } from 'next-safe-action/hooks'
import { createLeadAction } from '@/app/actions/leads'
import { createLeadSchema, type CreateLeadValues } from '@/lib/schemas/leads'
import {
  Form, FormField, FormItem, FormLabel, FormControl, FormMessage,
} from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'

export function CreateLeadForm({ onSuccess }: { onSuccess?: () => void }) {
  const form = useForm<CreateLeadValues>({
    resolver: zodResolver(createLeadSchema),
    defaultValues: { nome: '', telefone: '', email: '' },
  })

  const { execute, status } = useAction(createLeadAction, {
    onSuccess: ({ data }) => {
      if (data?.serverError) {
        // Erros server-side de volta para campos específicos
        Object.entries(data.serverError).forEach(([field, message]) => {
          form.setError(field as keyof CreateLeadValues, {
            type: 'server',
            message: message as string,
          })
        })
        return
      }
      form.reset()
      onSuccess?.()
    },
    onError: () => {
      // Erro genérico não mapeado a campo
      form.setError('root.serverError', {
        type: 'server',
        message: 'Erro interno. Tente novamente.',
      })
    },
  })

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit((data) => execute(data))} className="space-y-4">
        <FormField
          control={form.control}
          name="nome"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Nome</FormLabel>
              <FormControl><Input placeholder="Nome completo" {...field} /></FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="telefone"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Telefone</FormLabel>
              <FormControl><Input placeholder="(11) 99999-9999" {...field} /></FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Erro root (não mapeado a campo) */}
        {form.formState.errors.root?.serverError && (
          <p className="text-sm text-destructive">
            {form.formState.errors.root.serverError.message}
          </p>
        )}

        <Button
          type="submit"
          disabled={status === 'executing' || form.formState.isSubmitting}
          className="w-full"
        >
          {status === 'executing' ? 'Salvando...' : 'Criar Lead'}
        </Button>
      </form>
    </Form>
  )
}
```

### Quando usar `useFormStatus` vs `status` do next-safe-action

| Situação | Usar |
|----------|------|
| Desabilitar botão durante round-trip do servidor | `status === 'executing'` |
| Loading indicator fora do form component | `status` ou `isPending` do `useAction` |
| Formulário com `<form action={serverAction}>` sem RHF | `useFormStatus()` do React DOM |
| Validação client-side em andamento | `form.formState.isSubmitting` |

**Regra:** com RHF + next-safe-action, combinar `status === 'executing' || form.formState.isSubmitting` para `disabled` do botão.

---

## RBAC — 3 Camadas

### Tipos e hierarquia

```ts
// lib/rbac/types.ts
export type Role = 'admin' | 'gestor' | 'operador' | 'visualizador'

export const ROLE_HIERARCHY: Record<Role, number> = {
  admin: 4,
  gestor: 3,
  operador: 2,
  visualizador: 1,
}

export function hasMinRole(userRole: Role, minRole: Role): boolean {
  return ROLE_HIERARCHY[userRole] >= ROLE_HIERARCHY[minRole]
}
```

### Camada 1 — Middleware (edge)

```ts
// middleware.ts
import { createServerClient } from '@supabase/ssr'
import { hasMinRole, type Role } from '@/lib/rbac/types'

const ROUTE_RULES: Array<{ prefix: string; minRole: Role }> = [
  { prefix: '/(app)/admin', minRole: 'admin' },
  { prefix: '/(app)/financeiro', minRole: 'gestor' },
  { prefix: '/(app)/crm', minRole: 'operador' },
]

export async function middleware(req: NextRequest) {
  // ... criar supabaseMiddlewareClient ...
  const { data: { session } } = await supabase.auth.getSession()
  const rule = ROUTE_RULES.find(({ prefix }) => req.nextUrl.pathname.startsWith(prefix))

  if (rule) {
    if (!session) return NextResponse.redirect(new URL('/login', req.url))
    const role = session.user.app_metadata?.role as Role | undefined
    if (!role || !hasMinRole(role, rule.minRole)) {
      return NextResponse.redirect(new URL('/sem-acesso', req.url))
    }
  }
  return NextResponse.next()
}
```

### Camada 2 — RSC (re-verificação server-side)

```tsx
// app/(app)/admin/page.tsx
import { createClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

export default async function AdminPage() {
  const supabase = createClient()
  const { data: { session } } = await supabase.auth.getSession()

  if (!session) redirect('/login')
  const role = session.user.app_metadata?.role as Role
  if (!hasMinRole(role, 'admin')) redirect('/sem-acesso')

  // Só agora busca dados sensíveis
  const users = await fetchAdminUsers()
  return <AdminUserTable users={users} />
}
```

```ts
// Server Actions — mesma verificação
'use server'
export async function deleteLeadAction(leadId: string) {
  const supabase = createClient()
  const { data: { session } } = await supabase.auth.getSession()
  const role = session?.user.app_metadata?.role as Role
  if (!role || !hasMinRole(role, 'gestor')) throw new Error('Unauthorized')
  // ...
}
```

### Camada 3 — PermissionGate (UX, não segurança)

```tsx
// components/rbac/permission-gate.tsx
'use client'
import { useUser } from '@/lib/hooks/use-user'
import { hasMinRole, type Role } from '@/lib/rbac/types'

type Props = {
  minRole?: Role
  exactRoles?: Role[]
  fallback?: React.ReactNode
  children: React.ReactNode
}

export function PermissionGate({ minRole, exactRoles, fallback = null, children }: Props) {
  const { role } = useUser()
  if (!role) return <>{fallback}</>

  const allowed = minRole
    ? hasMinRole(role, minRole)
    : exactRoles ? exactRoles.includes(role) : true

  return allowed ? <>{children}</> : <>{fallback}</>
}

// lib/hooks/use-user.ts
'use client'
export function useUser() {
  const [role, setRole] = useState<Role | null>(null)
  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setRole(session?.user.app_metadata?.role as Role ?? null)
    })
  }, [])
  return { role }
}
```

```tsx
// Uso em componentes
<PermissionGate minRole="gestor">
  <Button variant="destructive">Excluir</Button>
</PermissionGate>

<PermissionGate exactRoles={['admin']} fallback={<p className="text-muted-foreground">Sem acesso</p>}>
  <AdminPanel />
</PermissionGate>
```

### Supabase Custom Claims — injetar role no JWT

```sql
-- Supabase Auth Hook (SQL function)
CREATE OR REPLACE FUNCTION custom_access_token_hook(event jsonb)
RETURNS jsonb AS $$
DECLARE user_role text;
BEGIN
  SELECT role INTO user_role FROM user_profiles WHERE id = (event->>'user_id')::uuid;
  RETURN jsonb_set(event, '{claims,app_metadata,role}',
    to_jsonb(COALESCE(user_role, 'visualizador')));
END;
$$ LANGUAGE plpgsql STABLE;
```

Registrar em Supabase Dashboard → Authentication → Hooks → Custom Access Token.

---

## Fontes
- https://react-hook-form.com/docs
- https://zod.dev
- https://ui.shadcn.com/docs/components/form
- https://next-safe-action.dev/docs/getting-started
- https://supabase.com/docs/guides/auth/custom-claims-and-role-based-access-control-rbac
