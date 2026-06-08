---
domain: n8n
category: knowledge
name: TEST_PATTERNS
title: "n8n — Test Patterns and Organization"
description: How n8n organizes, writes, and runs tests — unit, integration, and E2E
keywords: [n8n, tests, jest, vitest, playwright, unit, integration, e2e, coverage, fast-check, testcontainers, "persona:n8n-dev"]
updated: 2026-06-08
---

# n8n — Test Patterns and Organization

---

## Test Frameworks

n8n uses **two** test frameworks side by side:

| Framework | Where used | Config |
|-----------|------------|--------|
| **Jest** (via ts-jest) | Backend packages (`packages/cli`, `packages/core`, `packages/workflow`, `packages/nodes-base`) | `jest.config.js` per package |
| **Vitest** | Frontend packages (`packages/frontend/**`) and some `@n8n` packages | `vitest.workspace.ts` at root |
| **Playwright** | E2E (full browser + running n8n instance) | `packages/testing/playwright` / `n8n-playwright` filter |

---

## Directory Organization

### Backend packages (Jest)

Tests live in `__tests__/` subdirectories adjacent to source:
```
packages/core/src/
├── credentials.ts
├── __tests__/
│   └── credentials.test.ts
├── binary-data/
│   └── __tests__/
│       └── ...
```

`packages/cli` is different — it uses a top-level `test/` directory:
```
packages/cli/
├── src/
└── test/
    ├── unit/
    │   └── *.test.ts
    └── integration/
        └── *.test.ts
```

Test file naming: `<source-file>.test.ts` or `<source-file>.spec.ts`.

### Frontend packages (Vitest)
```
packages/frontend/editor-ui/
└── src/
    └── components/
        ├── SomeComponent.vue
        └── __tests__/
            └── SomeComponent.test.ts
```

---

## Running Tests

### Unit tests (fast, no DB)
```bash
# All packages
pnpm test:ci:backend:unit

# Specific package
pnpm --filter packages/cli test:unit
pnpm --filter packages/core run test

# Single test file
pnpm --filter packages/cli exec jest test/unit/foo.test.ts
```

### Integration tests (require DB)
```bash
# All integration tests
pnpm test:ci:backend:integration

# Specific package
pnpm --filter packages/cli test:integration

# With Testcontainers (isolated Docker DB)
pnpm --filter packages/cli exec jest --config jest.config.integration.testcontainers.js
```

### Frontend tests (Vitest)
```bash
pnpm test:ci:frontend
pnpm --filter packages/frontend/editor-ui run test
```

### E2E (Playwright — requires running n8n instance)
```bash
pnpm --filter=n8n-playwright test:local          # headless
pnpm --filter=n8n-playwright test:local --ui     # interactive
pnpm --filter=n8n-playwright test:local --grep="test name"  # filter
pnpm test:with:docker                             # CI-style with Docker
```

### Coverage
```bash
COVERAGE_ENABLED=true pnpm test
```

---

## Jest Configuration Details

Root `jest.config.js` uses `ts-jest` with `isolatedModules: true` (transpile-only, no full type check).

Per-package Jest variants in `packages/cli`:
| Config file | Purpose |
|-------------|---------|
| `jest.config.unit.js` | Unit tests only |
| `jest.config.integration.js` | Integration tests (real DB connection) |
| `jest.config.integration.testcontainers.js` | Integration with isolated Testcontainers DB |
| `jest.config.migration.js` | DB migration tests |

---

## What Good Tests Look Like in n8n

### Unit tests (Jest)
Standard `describe/it` with Jest matchers. Mock external dependencies with `jest.fn()`.

```typescript
describe('CredentialService', () => {
  it('should return decrypted credential', async () => {
    const service = new CredentialService(mockRepo, mockEncryption);
    const result = await service.getDecrypted(credentialId);
    expect(result.data).toEqual({ apiKey: 'secret' });
  });
});
```

### Property-based tests (fast-check)
The `packages/workflow` package uses `fast-check` for property-based testing:

```typescript
import { fc, test } from '@fast-check/jest';

test.prop([fc.array(fc.string())])('deduplicated array has no duplicates', (arr) => {
  const result = getConnectedNodes(arr);
  expect(new Set(result).size).toBe(result.length);
});
```

Reviewers increasingly ask for property tests when the behavior must hold for all inputs. See PR #31793 — "Dedupe getConnectedNodes results and add fast-check property tests."

### Integration tests
Use real DB via TypeORM + SQLite in-memory, or Testcontainers for PostgreSQL:

```typescript
describe('UserRepository (integration)', () => {
  let module: TestingModule;
  beforeAll(async () => {
    module = await Test.createTestingModule({...}).compile();
  });
  afterAll(() => module.close());
});
```

---

## Test Rules (from CONTRIBUTING.md + review patterns)

1. **No tests = unacceptable.** "A bug is not fixed unless a test prevents regression. A feature is not complete without tests." Community PRs without tests are auto-closed after 14 days.

2. **Unit tests for logic, integration for data access.** Don't mock the DB in integration tests — use real TypeORM with SQLite or Testcontainers.

3. **E2E for user-visible flows.** Playwright for workflows that exercise the UI and backend together.

4. **Test isolation.** Each test should clean up after itself. Use `beforeEach`/`afterEach`. Don't rely on test ordering.

5. **Property tests for invariants.** When a function must hold for all valid inputs, use `fast-check`. The team has started using it in `packages/workflow`.

6. **`How to test` section in the PR.** The PR template requires explaining how to test. For backend changes: exact `jest --testPathPattern=...` command. For frontend: browser steps. For node changes: example workflow.

---

## Coverage Expectations

- No hard coverage threshold is enforced in CI (not seen in jest configs).
- Reviewers notice when obvious paths are untested.
- The team runs mutation testing nightly (`mutation-health-nightly.yml`) and tracks a `.code-health-baseline.json`.

---

## See Also

- `BUILD_ENV` — test runner commands and CI workflow details
- `PROJECT_STACK` — which packages use which test frameworks
