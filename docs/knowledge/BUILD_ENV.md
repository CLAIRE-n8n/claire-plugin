---
domain: n8n
category: knowledge
name: BUILD_ENV
title: "n8n — Build System and Dev Environment"
description: How to set up, build, run, and test n8n locally — commands, tooling, and caveats
keywords: [n8n, build, pnpm, turbo, dev, env, setup, commands, biome, eslint, prettier, "persona:n8n-dev"]
updated: 2026-06-08
---

# n8n — Build System and Dev Environment

---

## Prerequisites

| Tool | Version | Notes |
|------|---------|-------|
| Node.js | ≥ 24 | Enforced via .nvmrc / engine check |
| pnpm | ≥ 10.22 | Via `corepack enable && corepack use pnpm@latest` |
| Docker | Any recent | Required for integration tests and E2E |

**⚠️ Do NOT use `npm install` or `yarn install` — the repo blocks it via `preinstall`.**

---

## Setup

```bash
# 1. Install dependencies
pnpm install

# 2. Build all packages
pnpm build

# 3. Set up local environment
cp .env.local.example .env.local
# Edit .env.local as needed
```

---

## Dev Commands

| Command | What it does |
|---------|-------------|
| `pnpm dev` | Start everything in watch mode (excludes design-system, chat, task-runner) |
| `pnpm dev:be` | Backend only (excludes frontend) |
| `pnpm dev:fe` | Frontend only (design-system + editor-ui) |
| `pnpm dev:ai` | AI nodes + core + CLI |
| `pnpm start` | Run the built app |
| `pnpm build` | Full build (Turbo, respects cache) |
| `pnpm build:unchecked` | Build without type checking (fast) |
| `pnpm typecheck` | TypeScript type check across all packages |
| `pnpm watch` | Alias for dev with max concurrency |

### Hot-reload for node development
```bash
N8N_DEV_RELOAD=true pnpm dev
```

### Per-package dev (faster for focused work)
```bash
cd packages/cli && pnpm dev       # backend only
cd packages/frontend/editor-ui && pnpm dev  # frontend only
```

---

## Build System: Turbo

n8n uses [Turborepo](https://turbo.build/) for incremental builds. Key behaviors:
- **Caching**: Turbo caches `dist/**` outputs per-package. Re-run `pnpm build` — only changed packages rebuild.
- **Dependency order**: `build` depends on `^build` (all dependencies must build first).
- **Force rebuild**: `TURBO_FORCE=true pnpm build` bypasses cache.
- **Affected builds**: `pnpm build --affected` builds only packages changed vs the default branch.

---

## Linting and Formatting

### ESLint (v9 flat config)
```bash
pnpm lint             # lint all packages
pnpm lint:fix         # auto-fix
pnpm lint:affected    # only changed packages
```

Each package has its own `eslint.config.mjs` extending `@n8n/eslint-config`.

**Key enforced rules:**
- `unicorn/filename-case: kebabCase` — all files must be kebab-case (**error**, not warning)
- `@typescript-eslint/no-explicit-any` — warn (tech debt, not blocking)
- `import-x/no-cycle` — warn (import cycles allowed for now but tracked)
- `n8n-local-rules/*` — custom rules for n8n-specific patterns

### Biome (formatter only, linter disabled)
```bash
turbo run format      # format all packages via Biome
turbo run format:check  # check without writing
```

Biome and Prettier share the same config: **tabs** (not spaces), single quotes, semicolons, `trailingComma: "all"`, 100-char line width.

### Prettier (Vue files only)
Biome excludes `**/*.vue` — Prettier handles Vue component formatting.

---

## TypeScript Configuration

Root `tsconfig.json` extends `packages/@n8n/typescript-config/tsconfig.common.json`.

Key settings from the shared config:
- `strict: true` — strict mode enabled
- `noUncheckedIndexedAccess: true`
- `noImplicitReturns: true`
- `isolatedModules: true` — in test configs (faster, no cross-file inference)

Each package has its own `tsconfig.json` + `tsconfig.build.json` (excludes tests for production builds).

---

## Git Hooks

Uses `lefthook` (not husky). Hooks run on commit and push:
```bash
# lefthook.yml controls what runs
pnpm lefthook run pre-commit   # manual trigger
```

Common pre-commit checks: lint, typecheck on changed files, format check.

---

## Environment Variables

Copy `.env.local.example` → `.env.local`. Prefix commands with `dotenvx` to load:
```bash
dotenvx run -- pnpm dev
```

Key env vars:
- `DB_TYPE` — `sqlite` (default, for local dev) or `postgresdb`
- `N8N_DEV_RELOAD` — enable hot reload for nodes
- `N8N_LOG_LEVEL` — `debug` | `info` | `warn` | `error`
- `COVERAGE_ENABLED` — enable Jest coverage collection

---

## CI Workflows

The repo has 70+ GitHub Actions workflows. Key ones for contributors:

| Workflow | Trigger | What it checks |
|----------|---------|----------------|
| `ci-pull-requests.yml` | PR | Unit + integration tests, lint, typecheck |
| `ci-check-pr-title.yml` | PR | Enforces conventional commit format |
| `ci-pr-quality.yml` | PR | Code quality gates |
| `test-e2e-reusable.yml` | PR/merge | Playwright E2E |
| `ci-cla-check.yml` | PR | CLA signature verification |
| `mutation-health-nightly.yml` | Nightly | Mutation testing |

**CI enforces PR title format** — the `ci-check-pr-title.yml` workflow will fail if the title doesn't follow `type(scope): Summary` convention.

---

## See Also

- `PROJECT_STACK` — package layout and architecture boundaries
- `TEST_PATTERNS` — test organization and commands
