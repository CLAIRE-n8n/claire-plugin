---
domain: n8n
category: knowledge
name: PROJECT_STACK
title: "n8n — Project Stack Overview"
description: Technology stack, package layout, and module boundaries for n8n-io/n8n
keywords: [n8n, stack, packages, typescript, vue, pnpm, turbo, monorepo]
updated: 2026-06-08
---

# n8n — Project Stack Overview

`n8n-io/n8n` is a TypeScript monorepo (pnpm + Turbo). The stack is ~91% TypeScript,
~7% Vue, with JavaScript, SCSS, and Python for tooling.

---

## Top-Level Package Layout

```
n8n-io/n8n/
├── packages/
│   ├── cli/              — Backend CLI, REST API, webhook server; the main runnable
│   ├── core/             — Workflow execution engine, binary-data, credentials
│   │                       ⚠️  Contact n8n team before changing this package
│   ├── workflow/         — Shared workflow interfaces (frontend + backend)
│   ├── nodes-base/       — All built-in nodes (500+)
│   ├── node-dev/         — CLI for scaffolding new nodes
│   ├── extensions/       — Plugin extension layer
│   ├── frontend/
│   │   ├── editor-ui/    — Vue workflow editor (main frontend)
│   │   └── @n8n/
│   │       ├── design-system/ — Vue component library
│   │       └── chat/          — Chat widget
│   └── @n8n/             — 38 scoped packages (see below)
├── packages/testing/     — Shared test utilities
└── scripts/              — Build, release, Docker scripts
```

### `packages/@n8n` Scoped Packages (38 packages)

Key packages:
| Package | Purpose |
|---------|---------|
| `@n8n/config` | Config schema (class-validator + @n8n/di) |
| `@n8n/db` | Database layer (TypeORM entities, migrations) |
| `@n8n/di` | Dependency injection container |
| `@n8n/errors` | Shared error hierarchy |
| `@n8n/api-types` | OpenAPI types shared across layers |
| `@n8n/permissions` | Scoped permission checking |
| `@n8n/nodes-langchain` | LangChain AI agent nodes |
| `@n8n/task-runner` | Sandboxed JS execution environment |
| `@n8n/expression-runtime` | n8n expression evaluator |
| `@n8n/eslint-config` | Shared ESLint configuration |
| `@n8n/typescript-config` | Shared tsconfig.common.json |
| `@n8n/vitest-config` | Shared Vitest configuration |

---

## Language Stack

| Language | % of codebase | Where |
|----------|---------------|-------|
| TypeScript | ~91% | All backend + shared packages |
| Vue (SFC) | ~7% | `packages/frontend/editor-ui`, `packages/frontend/@n8n/design-system` |
| JavaScript | <1% | Build scripts, legacy shims |
| SCSS | <1% | Frontend styling |
| Python | <1% | `scripts/`, `packages/testing/` utilities |

---

## Key Architecture Boundaries

1. **`packages/workflow`** — The common interface layer. Frontend and backend both import this. Keep it dependency-light.
2. **`packages/core`** — Execution engine. Only `packages/cli` should import from it. No direct imports from frontend.
3. **`packages/cli`** — Runs everything. Imports core, workflow, db, config. The HTTP API lives here.
4. **`packages/frontend`** — Never imports backend packages. Communicates only via the REST API.
5. **`packages/nodes-base`** — Node implementations. Each node is self-contained.

---

## Versioning

- **pnpm-workspace.yaml** uses a `catalog:` section to pin all dependency versions monorepo-wide.
- TypeScript: `6.0.2` (pinned)
- ESLint: `9.29.0` (flat config, pinned)
- Vitest: `^4.1.1`

---

## See Also

- `BUILD_ENV` — build commands, dev environment setup
- `TEST_PATTERNS` — test framework and organization
