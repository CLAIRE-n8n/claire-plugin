---
domain: n8n
category: knowledge
name: ARCHITECTURE
title: "n8n — Architecture and Layer Boundaries"
description: Package map, layer boundaries, key interfaces, and review-critical zones in n8n-io/n8n
keywords: [n8n, architecture, packages, layers, boundaries, interfaces, core, cli, workflow, frontend, nodes, dependency, "persona:n8n-dev", "persona:n8n-reviewer"]
updated: 2026-06-08
---

# n8n — Architecture and Layer Boundaries

---

## Package Map

```
n8n-io/n8n/
│
├── packages/cli/                     ← HTTP server, REST API, CLI entrypoint
│     └── src/
│         ├── executions/             ← Execution persistence, runner
│         ├── workflows/              ← Workflow activation, scheduling
│         ├── credentials/            ← Credential storage + access
│         ├── services/               ← Business services (projects, users, …)
│         ├── controllers/            ← HTTP route handlers
│         └── databases/              ← TypeORM entities, migrations
│
├── packages/core/                    ← Workflow execution engine ⚠️ RESTRICTED
│     └── src/
│         ├── execution-engine/       ← WorkflowExecute, run cycle
│         ├── node-execution-context/ ← Per-node execution context
│         └── binary-data/            ← Binary data backend
│
├── packages/workflow/                ← Shared interfaces (FE + BE)
│     └── src/
│         ├── interfaces.ts           ← INodeType, IWorkflowExecuteData, …
│         └── expression.ts           ← n8n expression language
│
├── packages/nodes-base/              ← Built-in nodes (500+)
│     └── nodes/
│         └── <NodeName>/
│             ├── <NodeName>.node.ts  ← Node implementation
│             └── <NodeName>.ts       ← (alternative entry)
│
├── packages/@n8n/
│   ├── db/                           ← TypeORM entities, migrations
│   ├── config/                       ← Config schema (class-validator)
│   ├── di/                           ← Dependency injection container
│   ├── nodes-langchain/              ← LangChain AI nodes
│   ├── task-runner/                  ← Sandboxed JS execution (Daytona)
│   ├── agents/                       ← AI agent runtime
│   ├── instance-ai/                  ← Daytona sandbox lifecycle
│   ├── api-types/                    ← OpenAPI shared types
│   ├── permissions/                  ← Scoped permission checking
│   └── …38 total scoped packages
│
└── packages/frontend/
    ├── editor-ui/                    ← Vue workflow editor
    └── @n8n/
        ├── design-system/            ← Vue component library
        └── chat/                     ← Chat widget
```

---

## Layer Boundaries (strict)

```
frontend/editor-ui
      │ REST API only
      ▼
packages/cli          ← Orchestration layer (imports all below)
      │
      ├──▶ packages/core          ← Execution engine (restricted)
      │         └──▶ packages/workflow
      ├──▶ packages/@n8n/db       ← Data persistence
      ├──▶ packages/@n8n/config   ← Config
      └──▶ packages/nodes-base    ← Node implementations
                └──▶ packages/workflow
```

### Rules

1. **Frontend never imports backend packages** — communicates via REST API only.
2. **`packages/workflow`** is the common interface layer. Both frontend and backend import it. Keep it dependency-light — no circular deps, no NestJS.
3. **`packages/core`** is imported only by `packages/cli`. No direct frontend import.
4. **`packages/nodes-base`** nodes are self-contained — each node in its own directory, no cross-node imports.
5. **`packages/@n8n/db`** is imported only by `packages/cli` (and test utilities).

---

## Key Interfaces

### INodeType (packages/workflow)

```typescript
export interface INodeType {
  description: INodeTypeDescription;
  execute?(this: IExecuteFunctions, items: INodeExecutionData[]): Promise<INodeExecutionData[][]>;
  trigger?(this: ITriggerFunctions): Promise<ITriggerResponse | undefined>;
  webhook?(this: IWebhookFunctions): Promise<IWebhookResponseData>;
  poll?(this: IPollFunctions): Promise<INodeExecutionData[][] | null>;
}
```

### IExecuteFunctions (packages/workflow)

The context object passed to node `execute()`. Key methods:
- `getInputData()` — returns the input items for the current node
- `getNodeParameter(name, index)` — reads a node parameter value
- `getCredentials(type)` — loads and decrypts credentials
- `helpers.request(options)` — makes HTTP requests (preferred over raw axios/fetch)

### WorkflowExecute (packages/core)

Core execution class. Handles:
- Node scheduling and queue
- Error handling + retry
- Binary data routing
- Execution state persistence

---

## Review-Critical Zones

| Zone | Package | Risk | Why |
|------|---------|------|-----|
| Execution engine | `packages/core` | HIGH | Bugs here affect all workflow runs |
| Sandbox lifecycle | `packages/@n8n/instance-ai/src/workspace/daytona-sandbox.ts` | HIGH | Async state machine; concurrent-call safety |
| AI agent runtime | `packages/@n8n/agents/src/runtime/agent-runtime.ts` | HIGH | Complex async orchestration |
| Execution persistence | `packages/cli/src/executions/execution-persistence.ts` | HIGH | Most-reviewed file (11 inline comments) |
| Credential access | `packages/cli/src/credentials/` | HIGH | Security-sensitive |
| REST API controllers | `packages/cli/src/controllers/` | MEDIUM | API contract |
| Frontend AI features | `packages/frontend/editor-ui/src/features/ai/` | MEDIUM | Rapid evolution |

**Before touching `packages/core`:** Read CONTRIBUTING.md — "Contact n8n before making changes to `packages/core`."

---

## Naming Conventions

| Item | Convention | Enforced by |
|------|-----------|-------------|
| File names | `kebab-case.ts` | ESLint error |
| TypeScript classes | `PascalCase` | Implicit |
| Functions / variables | `camelCase` | Implicit |
| Constants | `UPPER_SNAKE_CASE` | Implicit |
| Test files | `<name>.test.ts` in `__tests__/` | Implicit |
| Node class names | `<NodeName>` (PascalCase) | Convention |
| Node file names | `<NodeName>.node.ts` | Convention |

---

## Dependency Injection

`packages/@n8n/di` provides the IoC container. Services are decorated with `@Service()` and resolved by the container at runtime. `packages/cli/src/main.ts` bootstraps the DI container.

---

## Database

TypeORM + SQLite (dev) / PostgreSQL (production). Entities live in `packages/@n8n/db/src/entities/`. Migrations live in `packages/@n8n/db/src/migrations/`. Every schema change requires a migration — never modify entity files without one.

---

## See Also

- `PROJECT_STACK` — top-level package layout with dependency map
- `BUILD_ENV` — build commands, dev environment setup
- `TEST_PATTERNS` — test framework and how to run tests
