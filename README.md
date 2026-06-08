# CLAIRE-n8n/claire-plugin

Claire AI plugin for `n8n-io/n8n` — dev and reviewer personas.

---

## What This Is

This plugin teaches the [C.L.A.I.R.E.](https://github.com/claire-labs/claire) AI agent system
how to contribute to and review `n8n-io/n8n`. It provides:

- **Dev persona** (`n8n-dev`) — knows n8n's TypeScript monorepo, contribution conventions, PR
  format rules, test discipline, and common review blockers
- **Reviewer persona** (`n8n-reviewer`) — stateless gatekeeper that checks PR format, CI status,
  tests, architecture compliance, and n8n-specific review patterns

---

## How Claire Uses This

When claire is asked to work on an n8n issue, it:

1. Boots the `n8n-dev` persona (from `docs/persona/n8n-dev.md`)
2. Loads knowledge docs via `claire context persona:n8n-dev`
3. Implements the change in a worktree branch
4. Creates a PR following n8n's conventions
5. Boots the `n8n-reviewer` persona to review the PR
6. Merges after approval

The plugin is registered in `~/.config/claire/github_repos.yaml` so the claire
session-monitor can detect issue assignments and trigger the pipeline automatically.

---

## Document Index

### Personas

| File | Purpose |
|------|---------|
| `docs/persona/n8n-dev.md` | Developer agent — knows the full contribution workflow |
| `docs/persona/n8n-reviewer.md` | Reviewer agent — stateless PR gatekeeper |

### Knowledge

| File | Purpose |
|------|---------|
| `docs/knowledge/PROJECT_STACK.md` | Technology stack, package layout, module boundaries |
| `docs/knowledge/ARCHITECTURE.md` | Package map, layer boundaries, key interfaces |
| `docs/knowledge/BUILD_ENV.md` | Build system, dev commands, linting, CI workflows |
| `docs/knowledge/RUN_GUIDE.md` | How to run n8n locally for development |
| `docs/knowledge/TEST_PATTERNS.md` | Test frameworks, organization, and how to run tests |
| `docs/knowledge/REVIEW_PATTERNS.md` | Empirical review patterns mined from 167 merged PRs |

---

## Repository

- **Target repo:** `n8n-io/n8n` (or fork `CLAIRE-n8n/n8n`)
- **Plugin repo:** `CLAIRE-n8n/claire-plugin` (this repo)
- **Persona names:** `n8n-dev`, `n8n-reviewer`

---

## See Also

- [Claire V2 documentation](https://github.com/claire-labs/claire)
- [n8n contributing guide](https://github.com/n8n-io/n8n/blob/master/CONTRIBUTING.md)
