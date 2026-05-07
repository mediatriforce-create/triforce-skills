# Server Actions Security + RLS — Guia de Revisão

## Server Actions — Modelo Mental Obrigatório

**Server Actions são endpoints POST públicos.** Middleware protege a rota da página — não a action. Uma action sem auth check dentro dela é acessível por qualquer `fetch()` direto, mesmo que a página seja protegida.

```ts
// 🔴 BLOQUEANTE — "a página é protegida, então a action também é"
// FALSO. Qualquer um pode fazer POST direto para a action URL.
export const deleteLeadAction = actionClient
  .schema(z.object({ id: z.string() }))
  .action(async ({ parsedInput }) => {
    await db.delete(leads).where(eq(leads.id, parsedInput.id))
  })

// ✅ CORRETO — auth verificada DENTRO da action
export const deleteLeadAction = authedActionClient // cliente com .use(authMiddleware)
  .schema(z.object({ id: z.string() }))
  .action(async ({ parsedInput, ctx }) => {
    // ctx.user injetado pelo authMiddleware — se chegou aqui, está autenticado
    // mas autenticação ≠ autorização — verificar role
    if (!hasMinRole(ctx.user.role, 'gestor')) {
      throw new Error('Unauthorized')
    }
    await db.delete(leads).where(
      and(eq(leads.id, parsedInput.id), eq(leads.createdBy, ctx.user.id))
    )
  })
```

### next-safe-action NÃO auto-autentica

```ts
// lib/safe-action.ts

// 🔴 BLOQUEANTE — actionClient base é público
export const actionClient = createSafeActionClient()
// qualquer action usando este client é pública

// ✅ CORRETO — authedActionClient encapsula auth
export const authedActionClient = actionClient.use(async ({ next }) => {
  const supabase = await createClient()
  const { data: { user }, error } = await supabase.auth.getUser()
  if (error || !user) throw new Error('Unauthorized')
  return next({ ctx: { user } })
})

// ✅ CORRETO — rateLimitedActionClient encapsula ambos
export const rateLimitedActionClient = authedActionClient.use(async ({ next, ctx }) => {
  const ip = ipAddress(headers()) ?? 'anonymous'
  const { success } = await ratelimit.limit(ip)
  if (!success) throw new Error('Rate limit exceeded')
  return next({ ctx })
})
```

### Padrão DAL (Data Access Layer) com React cache()

```ts
// lib/dal.ts — deduplicar auth calls dentro do mesmo request
import { cache } from 'react'

export const verifySession = cache(async (): Promise<Session> => {
  const supabase = await createClient()
  const { data: { user }, error } = await supabase.auth.getUser()
  if (error || !user) redirect('/login')
  return { userId: user.id, role: user.app_metadata?.role as Role }
})

// Em qualquer Server Action ou RSC — cache() garante que roda só 1x por request
export const someAction = authedActionClient
  .action(async ({ ctx }) => {
    const { role } = await verifySession() // deduplicated — não faz novo round-trip
    // ...
  })
```

---

## RLS Supabase — Checklist de Revisão

### `withRLS()` vs `enableRLS()` — a diferença

```ts
// 🔴 BLOQUEANTE — enableRLS() deprecated desde Drizzle v1.0-beta.1
export const leads = pgTable('leads', {
  id: uuid('id').defaultRandom().primaryKey(),
  // ...
}).enableRLS() // DEPRECATED — não tem efeito garantido em v1.0

// ✅ CORRETO
export const leads = pgTable('leads', {
  id: uuid('id').defaultRandom().primaryKey(),
  // ...
}).withRLS()
```

### Verificar policies via MCP (execute_sql)

```sql
-- Verificar que RLS está ativado
SELECT tablename, rowsecurity FROM pg_tables
WHERE schemaname = 'public' AND tablename = 'leads';

-- Verificar policies existentes
SELECT policyname, cmd, roles, qual, with_check
FROM pg_policies
WHERE tablename = 'leads';

-- Verificar que políticas cobrem os 4 comandos (SELECT, INSERT, UPDATE, DELETE)
-- Se faltar algum → default deny para aquele comando
```

### auth.uid() NULL no Drizzle — gotcha crítico

```ts
// 🔴 BLOQUEANTE — Drizzle conecta via raw Postgres
// auth.uid() depende de 'request.jwt.claims' no contexto da sessão
// Drizzle não define isso por padrão → auth.uid() = NULL
// RLS policies que usam auth.uid() silenciosamente negam TUDO

const { db } = createDb()
const leads = await db.select().from(leadsTable)
// Se a policy é: auth.uid() = user_id → retorna [] sem erro

// ✅ CORRETO — definir jwt claims na transaction
export async function getLeadsForUser(userId: string, jwtClaims: string) {
  const { db, client } = createDb()
  try {
    return await db.transaction(async (tx) => {
      await tx.execute(
        sql`SELECT set_config('request.jwt.claims', ${jwtClaims}, true)`
      )
      return tx.select().from(leadsTable)
    })
  } finally {
    await client.end()
  }
}
```

### Views e security_invoker

```sql
-- 🔴 BLOQUEANTE — view executa sob contexto do criador (definer)
-- Bypassa RLS completamente se o criador tem acesso total
CREATE VIEW leads_view AS SELECT * FROM leads;

-- ✅ CORRETO — security_invoker garante que RLS do chamador é respeitado
CREATE VIEW leads_view WITH (security_invoker = true) AS SELECT * FROM leads;
```

### RLS habilitado mas sem policies = default-deny silencioso

```sql
-- Estado perigoso: RLS ON mas nenhuma policy
-- SELECT retorna 0 linhas sem erro — difícil de debugar

-- Verificar:
SELECT COUNT(*) FROM pg_policies WHERE tablename = 'leads';
-- Se 0 → ou está errado ou deveria ser 0 por design (confirmar com Gabriel)
```

---

## Perguntas obrigatórias em todo PR que toca Server Actions

1. O `actionClient` usado é `authedActionClient` ou o base `actionClient` (público)?
2. Há verificação de role (RBAC) dentro da action, além da autenticação?
3. Zod schema aplicado antes de qualquer operação no banco?
4. Se a action modifica dados de outro usuário, há verificação de ownership?

## Perguntas obrigatórias em todo PR que toca schema/queries Supabase

1. `.withRLS()` (não `.enableRLS()`) nas tabelas novas/modificadas?
2. Para queries via Drizzle com RLS: `set_config('request.jwt.claims', ...)` sendo chamado?
3. Views novas têm `security_invoker = true`?
4. Verificar via `execute_sql` que policies cobrem os 4 comandos na tabela afetada.
5. `prepare: false` na conexão se usando pooler Transaction (porta 6543)?

---

## Fontes
- https://nextjs.org/docs/app/guides/data-fetching/server-actions-and-mutations
- https://next-safe-action.dev/docs/safe-action-client/middleware
- https://supabase.com/docs/guides/database/postgres/row-level-security
- https://orm.drizzle.team/docs/rls
- https://owasp.org/www-project-top-ten/
