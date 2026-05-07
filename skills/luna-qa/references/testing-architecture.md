# Testing Architecture — triforce-sistema

## Setup Vitest Completo

```bash
npm install -D vitest @vitejs/plugin-react @testing-library/react @testing-library/dom \
  vite-tsconfig-paths @electric-sql/pglite next-test-api-route-handler \
  @vitest/coverage-v8
```

```ts
// vitest.config.ts
import { defineConfig } from "vitest/config"
import react from "@vitejs/plugin-react"
import tsconfigPaths from "vite-tsconfig-paths"

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    environment: "jsdom",
    setupFiles: ["./tests/setup.ts"],
    pool: "forks", // obrigatório para PGlite WASM
    coverage: {
      provider: "v8",
      reporter: ["text", "lcov", "html"],
      reportsDirectory: "./coverage",
      thresholds: {
        lines: 0,        // começa em 0, sobe via autoUpdate
        functions: 0,
        branches: 0,
        statements: 0,
        autoUpdate: true,
      },
    },
  },
})
```

```ts
// tests/setup.ts
import { vi } from "vitest"

// Mock next/navigation — usado por Server Actions com redirect
vi.mock("next/navigation", () => ({
  redirect: vi.fn(),
  notFound: vi.fn(),
}))

// Mock next/headers — usado por cookies() e headers()
vi.mock("next/headers", () => ({
  cookies: vi.fn(() => ({
    get: vi.fn(),
    set: vi.fn(),
    delete: vi.fn(),
  })),
  headers: vi.fn(() => new Headers()),
}))
```

---

## Setup Playwright Completo

```bash
npm install -D @playwright/test
npx playwright install --with-deps chromium
```

```ts
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test"

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",
  use: {
    baseURL: process.env.PLAYWRIGHT_TEST_BASE_URL || "http://localhost:3000",
    trace: "on-first-retry",
    storageState: "e2e/.auth/user.json", // auth persistido
  },
  projects: [
    {
      name: "setup",
      testMatch: /global.setup\.ts/,
    },
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
      dependencies: ["setup"],
    },
  ],
})
```

```ts
// e2e/global.setup.ts
import { chromium } from "@playwright/test"
import { createClient } from "@supabase/supabase-js"

async function globalSetup() {
  // Criar sessão real via Supabase
  const supabase = createClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY! // bypassa RLS para seed
  )

  // Login com usuário de teste
  const { data, error } = await supabase.auth.signInWithPassword({
    email: process.env.TEST_USER_EMAIL!,
    password: process.env.TEST_USER_PASSWORD!,
  })
  if (error || !data.session) throw new Error("Login de teste falhou")

  // Salvar cookies de sessão para reutilizar nos testes
  const browser = await chromium.launch()
  const context = await browser.newContext()
  await context.addCookies([
    {
      name: "sb-access-token",
      value: data.session.access_token,
      domain: "localhost",
      path: "/",
    },
    {
      name: "sb-refresh-token",
      value: data.session.refresh_token,
      domain: "localhost",
      path: "/",
    },
  ])
  await context.storageState({ path: "e2e/.auth/user.json" })
  await browser.close()
}

export default globalSetup
```

---

## Setup PGlite para Smoke Test de Migrations

```ts
// tests/migration.smoke.test.ts
import { PGlite } from "@electric-sql/pglite"
import { drizzle } from "drizzle-orm/pglite"
import { migrate } from "drizzle-orm/pglite/migrator"
import { describe, it, expect, beforeAll } from "vitest"

describe("migrations smoke test", () => {
  let db: ReturnType<typeof drizzle>

  beforeAll(async () => {
    const client = new PGlite()
    db = drizzle(client)
    // Aplica todas as migrations de ./supabase/migrations
    await migrate(db, { migrationsFolder: "./supabase/migrations" })
  })

  it("aplica todas as migrations sem erro", () => {
    expect(db).toBeDefined()
  })

  it("schema tem as tabelas esperadas", async () => {
    const result = await db.execute(
      `SELECT table_name FROM information_schema.tables WHERE table_schema = 'public' ORDER BY table_name`
    )
    const tables = result.rows.map((r: Record<string, unknown>) => r.table_name)
    expect(tables).toContain("leads")
    expect(tables).toContain("clientes")
    expect(tables).toContain("interacoes")
    expect(tables).toContain("transactions") // vai falhar até a migration ser criada
  })
})
```

---

## pgTAP — RLS Testing

```sql
-- supabase/tests/database/rls_leads.test.sql
BEGIN;
SELECT plan(4);

-- Helper: simular JWT de usuário autenticado
CREATE OR REPLACE FUNCTION set_auth(user_id uuid) RETURNS void AS $$
BEGIN
  PERFORM set_config('request.jwt.claims',
    json_build_object('sub', user_id, 'role', 'authenticated')::text, true);
  SET LOCAL ROLE authenticated;
END;
$$ LANGUAGE plpgsql;

-- Seed: criar lead pertencente ao user_A
INSERT INTO auth.users (id, email) VALUES ('user-a-id'::uuid, 'a@test.com');
INSERT INTO leads (id, nome, created_by) VALUES (gen_random_uuid(), 'Lead A', 'user-a-id'::uuid);

-- Teste 1: user_A vê seu próprio lead
SELECT set_auth('user-a-id'::uuid);
SELECT results_eq(
  'SELECT count(*)::int FROM leads',
  ARRAY[1],
  'user_A vê seu próprio lead'
);

-- Teste 2: user_B não vê lead do user_A
SELECT set_auth('user-b-id'::uuid);
SELECT results_eq(
  'SELECT count(*)::int FROM leads',
  ARRAY[0],
  'user_B não vê leads de user_A'
);

-- Teste 3: anon não vê nada
SET LOCAL ROLE anon;
SELECT results_eq(
  'SELECT count(*)::int FROM leads',
  ARRAY[0],
  'anon não vê leads'
);

-- Teste 4: service_role bypassa RLS (comportamento esperado)
RESET ROLE;
SELECT results_eq(
  'SELECT count(*)::int FROM leads',
  ARRAY[1],
  'service_role bypassa RLS corretamente'
);

SELECT * FROM finish();
ROLLBACK;
```

Rodar: `supabase test db`

---

## Fontes
- https://github.com/rphlmr/drizzle-vitest-pg
- https://playwright.dev/docs/auth
- https://supabase.com/docs/guides/local-development/testing/pgtap-extended
- https://vitest.dev/config/coverage
- https://www.npmjs.com/package/next-test-api-route-handler
