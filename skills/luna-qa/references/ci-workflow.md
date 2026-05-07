# CI Workflow — GitHub Actions (triforce-sistema)

## Workflow Completo (Vitest + Playwright + pgTAP)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    name: unit-tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npx vitest run --coverage
        env:
          NODE_ENV: test
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-report
          path: coverage/
          retention-days: 30

  migration-smoke:
    name: migration-smoke
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npx vitest run tests/migration.smoke.test.ts
        env:
          NODE_ENV: test

  db-tests:
    name: db-tests (pgTAP)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: supabase/setup-cli@v1
        with:
          version: latest
      - name: Start Supabase local
        run: supabase start
      - name: Run pgTAP RLS tests
        run: supabase test db
      - name: Stop Supabase
        run: supabase stop

  e2e-tests:
    name: e2e-tests
    runs-on: ubuntu-latest
    needs: [unit-tests, db-tests] # E2E só roda se unit + db passarem
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium
      - name: Build app
        run: npm run build
        env:
          NEXT_PUBLIC_SUPABASE_URL: ${{ secrets.NEXT_PUBLIC_SUPABASE_URL }}
          NEXT_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.NEXT_PUBLIC_SUPABASE_ANON_KEY }}
      - name: Run Playwright tests
        run: npx playwright test
        env:
          PLAYWRIGHT_TEST_BASE_URL: http://localhost:3000
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          SUPABASE_SERVICE_ROLE_KEY: ${{ secrets.SUPABASE_SERVICE_ROLE_KEY }}
          NEXT_PUBLIC_SUPABASE_URL: ${{ secrets.NEXT_PUBLIC_SUPABASE_URL }}
          NEXT_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.NEXT_PUBLIC_SUPABASE_ANON_KEY }}
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

## Restrição de Branch Protection (GitHub Free + repo privado)

**GitHub Free não permite `required status checks` em repositórios privados.**

Enforcement alternativo para equipe-sistemas:
1. CI que falha é visível no PR — developer não pode ignorar visualmente
2. Rodrigo tem regra explícita: **não revisar PR com CI falhando**
3. Luna tem autoridade de bloqueio — pode pedir que PR seja fechado e reaberto após correção
4. Convention de equipe: merge sem CI verde = violação de processo, não erro técnico

Se o repositório for tornado público no futuro, habilitar em Settings > Branches:
- "Require status checks to pass before merging"
- Jobs a adicionar: `unit-tests`, `migration-smoke`, `db-tests`, `e2e-tests`

---

## Variáveis de Ambiente para CI

Adicionar em GitHub > Settings > Secrets and variables > Actions:

| Secret | Descrição |
|--------|-----------|
| `NEXT_PUBLIC_SUPABASE_URL` | URL do projeto Supabase (pode ser local para CI) |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Chave anon do Supabase |
| `SUPABASE_SERVICE_ROLE_KEY` | Service role (para seed de dados de teste) |
| `TEST_USER_EMAIL` | Email do usuário de teste E2E |
| `TEST_USER_PASSWORD` | Senha do usuário de teste E2E |

**NUNCA** usar `SUPABASE_DB_URL` do projeto remoto em CI — sempre usar `supabase start` local.

---

## package.json scripts recomendados

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:db": "supabase test db",
    "test:all": "npm run test:run && npm run test:db && npm run test:e2e"
  }
}
```

---

## Fontes
- https://playwright.dev/docs/ci-intro
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches
- https://github.com/supabase/setup-cli
