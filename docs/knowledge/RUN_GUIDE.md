---
domain: n8n
category: knowledge
name: RUN_GUIDE
title: "n8n — Local Run Guide"
description: How to run n8n locally for development — startup, login, workflow creation, and environment options
keywords: [n8n, run, local, development, startup, login, workflow, debug, env, sqlite, postgres, "persona:n8n-dev"]
updated: 2026-06-08
---

# n8n — Local Run Guide

How to start n8n locally and verify it is working.

---

## Quick Start (first time)

```bash
# 1. Install dependencies
pnpm install

# 2. Build all packages
pnpm build

# 3. Copy local environment file
cp .env.local.example .env.local

# 4. Start everything in dev mode
pnpm dev
```

n8n starts at **http://localhost:5678** (default port).

---

## Login

On first run, n8n presents a setup wizard. Use any email + password — these are local-only, not sent anywhere.

After setup: **http://localhost:5678** → Login with your credentials.

---

## Dev Mode Variants

| Command | What runs | When to use |
|---------|-----------|-------------|
| `pnpm dev` | Full stack (BE + FE, watch mode) | Default for most work |
| `pnpm dev:be` | Backend only (no frontend rebuild) | Pure backend / API changes |
| `pnpm dev:fe` | Frontend + design-system only | UI / editor changes |
| `pnpm dev:ai` | AI nodes + core + CLI | AI agent node development |
| `N8N_DEV_RELOAD=true pnpm dev` | Full stack + hot-reload nodes | Node development |

---

## Useful Environment Variables

Set these in `.env.local` (prefix commands with `dotenvx run --` to load):

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_TYPE` | `sqlite` | Database backend: `sqlite` or `postgresdb` |
| `N8N_PORT` | `5678` | HTTP port |
| `N8N_LOG_LEVEL` | `info` | Log verbosity: `debug` \| `info` \| `warn` \| `error` |
| `N8N_DEV_RELOAD` | — | Set to `true` to enable hot-reload for node changes |
| `COVERAGE_ENABLED` | — | Set to `true` to enable Jest coverage |
| `N8N_RUNNERS_ENABLED` | — | Enable sandboxed task-runner mode |

```bash
# Load .env.local and start dev
dotenvx run -- pnpm dev

# Debug log level
N8N_LOG_LEVEL=debug pnpm dev:be
```

---

## Running with PostgreSQL

For integration testing or production-parity work:

```bash
# 1. Start Postgres (Docker)
docker run -d \
  --name n8n-postgres \
  -e POSTGRES_USER=n8n \
  -e POSTGRES_PASSWORD=n8n \
  -e POSTGRES_DB=n8n \
  -p 5432:5432 \
  postgres:16

# 2. Set env vars in .env.local
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=n8n

# 3. Start n8n
dotenvx run -- pnpm dev
```

---

## Running the Built App (no watch mode)

```bash
pnpm build
pnpm start
```

Use `pnpm start` for testing the production build locally.

---

## Accessing the API

n8n exposes a REST API at `http://localhost:5678/api/v1/`. Requires an API key:

1. In the n8n UI: **Settings → n8n API → Create an API Key**
2. Test: `curl -H "X-N8N-API-KEY: <key>" http://localhost:5678/api/v1/workflows`

Full API reference: `http://localhost:5678/api/swagger` (Swagger UI).

---

## Testing a Specific Node

```bash
# 1. Start dev mode with node hot-reload
N8N_DEV_RELOAD=true pnpm dev

# 2. Edit your node file in packages/nodes-base/nodes/<NodeName>/
#    The change reloads automatically

# 3. Create a test workflow in the UI:
#    - Add your node
#    - Set parameters
#    - Run with "Execute workflow"
```

---

## Running Tests (quick reference)

```bash
# Unit tests — specific package
pnpm --filter packages/cli test:unit

# Integration tests — requires Docker (DB + Redis)
pnpm --filter packages/cli test:integration

# E2E — requires full running n8n instance
pnpm --filter=n8n-playwright test:local
```

See `TEST_PATTERNS` for full test commands.

---

## Common Issues

### Port already in use
```bash
# Find what's on port 5678
lsof -i :5678
# Change port in .env.local
N8N_PORT=5679
```

### Build cache stale
```bash
TURBO_FORCE=true pnpm build
```

### Module not found after `pnpm install`
```bash
# Full clean reinstall
rm -rf node_modules packages/*/node_modules
pnpm install
pnpm build
```

### Database migration needed
```bash
# n8n auto-runs migrations on startup
# If you see migration errors, reset the SQLite DB:
rm ~/.n8n/database.sqlite
pnpm dev  # recreates on fresh start
```

---

## See Also

- `BUILD_ENV` — full build system reference (Turbo, ESLint, Biome, TypeScript config)
- `ARCHITECTURE` — package map and layer boundaries
- `TEST_PATTERNS` — test organization and commands
