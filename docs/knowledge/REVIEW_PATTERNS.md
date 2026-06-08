---
domain: n8n
category: knowledge
name: REVIEW_PATTERNS
title: "n8n — PR Review Patterns (mined from n8n-io/n8n)"
description: "Empirical review patterns extracted from the last 200 closed PRs in n8n-io/n8n. Approval signals, blocker signals, reviewer vocabulary, and merge criteria."
keywords: [n8n, review, patterns, approval, blockers, vocabulary, merge-criteria, reviewer, lgtm, changes-requested]
updated: 2026-06-07
---

# n8n — PR Review Patterns

Derived from 167 merged PRs and 430 reviews (206 inline comments) from
`n8n-io/n8n`, collected from the 200 most-recently-closed PRs as of 2026-06-07.

---

## 1. Review State Distribution

| State | Count | % |
|-------|-------|---|
| COMMENTED | 222 | 52% |
| APPROVED | 179 | 42% |
| DISMISSED | 18 | 4% |
| CHANGES_REQUESTED | 11 | 3% (of 430 total reviews) |

**Key insight:** The vast majority of reviews end in APPROVED or COMMENTED — not
CHANGES_REQUESTED. Blockers are uncommon. Most review back-and-forth happens via
inline comments while the overall review state is still COMMENTED or APPROVED.

---

## 2. Top Human Reviewers

| Reviewer | Reviews | Inline comments |
|----------|---------|-----------------|
| shortstacked | 62 | 16 |
| Flexicon | 20 | 11 |
| Cadiac | 17 | 14 |
| Matsuuu | 17 | 11 |
| mike12345567 | 13 | 9 |
| yehorkardash | 11 | 13 |
| aalises | 11 | — |
| yuliia-pominchuk | 11 | 6 |
| ivov | 8 | 6 |
| riqwan | 8 | — |

**shortstacked** is the de facto lead reviewer — highest volume, most
CHANGES_REQUESTED, and most thorough inline comments.

---

## 3. APPROVE Signals

### Vocabulary (LGTM comments with body, N=54)

**Short-form approvals:**
- `LGTM` / `lgtm!` / `LGTM!` / `LGTM 🚀` — most common
- `:feelsgood:` — used by shortstacked
- `🚀` — used by RicardoE105, tomi
- `👍` / `👍🏻` — used by Flexicon, tomi
- `lgtm` — used by sovietspaceship

**With qualification:**
- `"Just one nit comment, otherwise LGTM 👍"` — approves with non-blocking inline comment
- `"Couple of maintenance / documentation level nit's. Otherwise a really smart approach"` — COMMENTED state, not APPROVAL
- `"As always, splendid work. LGTM! 🚀"`
- `"Thanks for handling the comments! Full speed gogogo"`
- `"Really good idea on the template DB!"` — affirms design choice, then approves

**With manual verification note:**
- `"Tested locally, works nicely 👌"`
- `"Verified in stacked PR #31760"` — points to evidence
- `"Pins both node24/alpine3.22 bases … Full CI green incl. all 16 E2E shards"` — summarizes CI result

### What reviewers approve on

1. **Tests pass and CI is green** — most referenced signal
2. **Code style consistent** — Biome formatter passing, no `ts-ignore`
3. **TypeScript compliance** — no type errors, proper typing
4. **Small, focused PR** — single concern per PR
5. **Prior review comments addressed** — "Thanks for handling the comments"
6. **Architecture compliance** — no cross-layer imports, no duplicate exports
7. **Manual verification** — reviewer explicitly ran the change locally or in CI

---

## 4. CHANGES_REQUESTED Blockers (5 most common)

### Blocker 1: Tests fail or do not cover the change

**Signal:** Reviewer runs the PR locally and test step exits 1.

> "Tested this on a fresh checkout (`pnpm agent:setup`). Install + build pass,
> but the test step exits 1 — which means the script doesn't yet deliver on its
> core promise." — shortstacked on PR #31756

> "Point 3: Two tests are almost identical, pull setup into a helper or it.each.
> Point 4: Add a case with `null` and `number` or `boolean`
> are missing from the latest diff." — alexander-gekov on PR #31704

**Rule:** A PR promising "a fresh agent can verify a checkout" must have tests
that actually pass on a fresh checkout. Promise = test.

### Blocker 2: Formatting (Biome)

**Signal:** `biome ci .` fails with `File content differs from formatting output`.

> "Fix the formatting please" + CI log with exact diff — shortstacked on PR #31611

**Rule:** Run `pnpm format` before pushing. `pnpm format:check` is a CI gate.

### Blocker 3: Build fails on this branch

**Signal:** Reviewer attempts to build the devcontainer or a package and gets a
non-zero exit.

> "Pulled this down and tried to build the devcontainer image — the build fails
> on this branch." — shortstacked on PR #31653

**Rule:** Every PR must build cleanly: `pnpm install && pnpm build`.

### Blocker 4: Prior review comments not addressed

**Signal:** Second review cycle and the previous change requests are still missing.

> "It looks like the fix you mentioned for Point 3 and Point 4 are missing from
> the latest diff. Could you address those?" — alexander-gekov on PR #31704

**Rule:** Never push a "re-review request" without a commit addressing every
outstanding comment.

### Blocker 5: Overly complex mitigation / production import change

**Signal:** Fix stacks multiple mitigations where one would do, or changes
production import semantics outside the fix's scope.

> "The change stacks three mitigations where one per test would do, and one of
> them alters production import semantics outside this fix's scope." — shortstacked on PR #31592

**Rule:** Match the mitigation to the scope. Fix-the-test PRs must not change
production code paths.

---

## 5. COMMENTED Review Patterns (not blocking, but expected to be addressed)

### Pattern: nit — readability

- Rename variable/function when name doesn't match behavior
- Extract nested loops into a named helper
- Add a one-liner comment for non-obvious conventions (e.g. Playwright `use(undefined)` contract)

> "The readability of this triple-nested for loop and the fact that it just calls
> `idOf` without the function name actually explaining the side effect it's
> executing for is a bit confusing." — Matsuuu

**Response:** Either fix it (preferred) or explain why the current name/shape is intentional.

### Pattern: nit — test coverage scope

- Add a case for `null`, `number`, `boolean` (falsy / type boundaries)
- Pull identical test setups into `beforeEach` or `it.each`
- Add a focused test for the sandbox-disabled / already-initialized path

> "Would be good to have a test that checks simultaneous change and add case.
> For example, `[task-1, task-2]` -> `[task-2, task-3]`" — yehorkardash

### Pattern: architecture question

- "Do we need to export these from the barrel file too?"
- "Should we migrate `getWorkflowsExecutedSince(date)` to ExecutionPersistence?"

These are suggestions / open questions. Reviewer expects either a fix or a
reasoned "no, because X" in the thread.

### Pattern: error handling concern

- "This helper clears the `startPromise`, so two callers recovering at the same
  time could accidentally start two concurrent recoveries." — OlegIvaniv
- "`recoverAndRetry()` can now retry after any failed operation if the later state
  machine allows it." — OlegIvaniv

These are correctness questions about race conditions or partial-failure semantics.
Requires either a fix or an explanation of why the scenario cannot occur.

---

## 6. Reviewer Vocabulary Reference

### APPROVE templates

| Phrase | Context |
|--------|---------|
| `LGTM` / `lgtm!` | Standard, minimal |
| `🚀` | Enthusiastic, team members |
| `:feelsgood:` | shortstacked-specific |
| `"Thanks for handling the comments! Full speed gogogo"` | After addressing feedback |
| `"Tested locally, works nicely 👌"` | After manual run |
| `"Verified in stacked PR #N"` | When verification is in another PR |
| `"As always, splendid work. LGTM! 🚀"` | For recurring contributors |
| `"Absolute fantastic addition, thanks @user ❤️"` | For significant new features |

### CHANGES_REQUESTED opener templates

| Phrase | Meaning |
|--------|---------|
| `"Tested this on a fresh checkout … but the test step exits 1"` | Build/test verification failed |
| `"Fix the formatting please"` + biome diff | Biome CI gate failure |
| `"It looks like the fix you mentioned for X is missing from the latest diff"` | Prior comment not addressed |
| `"The build fails on this branch"` + error | Build gate failure |
| `"Diagnosis checks out … but the change stacks three mitigations"` | Over-scoped fix |

### Nitpick openers (inline, non-blocking)

| Phrase | Meaning |
|--------|---------|
| `"nit:"` | Cosmetic / minor style |
| `"Could we …?"` | Suggestion open to discussion |
| `"I wonder if …"` | Design question, not mandatory |
| `"WDYT"` | What do you think — genuine question |
| `"optional:"` | Take it or leave it |
| `"feel free to disagree"` | Opinion, not a blocker |

---

## 7. Merge Criteria

### Required for every PR

1. **`cla-signed` label** — present on all 167 merged PRs. Added automatically by
   the CLA bot when the contributor signs the CLA.
2. **CI green** — `CI: Pull Requests (Build, Test, Lint)` must pass.
3. **At least 1 human APPROVED review** — most merged PRs have 1–2 approvals.
4. **PR title follows Angular convention** — validated by `ci-check-pr-title.yml`.
   Format: `<type>(<scope>): <Summary>`. Types: `feat`, `fix`, `perf`, `test`,
   `docs`, `refactor`, `build`, `ci`, `chore`.

### Common labels observed

| Label | Meaning |
|-------|---------|
| `cla-signed` | CLA signed — required |
| `n8n team` | Author is n8n team member |
| `core` | Changes to `packages/core` or `packages/cli` |
| `Backport to Beta` / `Backport to Stable` | Backport required to release branch |
| `automation:backport` | Backport automation triggered |
| `Released` | Already released |
| `release` + `release:patch` | Release PR |

### Min approvals

Based on observed merges: **1 approval** is the minimum for team PRs. Community
PRs follow the same rule but with stricter blocker response time (14 days).

### Merge method

Squash merge is the standard (single commit per PR in `master` history).

---

## 8. Automated Reviewers

Two automated review bots post on every PR:

| Bot | What it does |
|-----|-------------|
| `cubic-dev-ai[bot]` | Architecture diagram + issue count (e.g. "1 issue found across 8 files"). Link to fix on cubic.dev. |
| `n8n-release-helper[bot]` | Automates backport and release labeling. |

**Ignore `cubic-dev-ai` findings unless they point to a real blocker.** Human
reviewers do not reference cubic-dev-ai findings in their approvals.

---

## 9. Most Reviewed Files / Areas

Based on inline comment distribution, these files attract the most scrutiny:

| File / Area | Comments | Why |
|-------------|----------|-----|
| `packages/cli/src/executions/execution-persistence.ts` | 11 | Core execution logic |
| `packages/cli/src/services/project.service.ee.ts` | 9 | EE project service |
| `packages/core/src/execution-engine/workflow-execute.ts` | 8 | Core execution engine |
| `packages/@n8n/agents/src/runtime/agent-runtime.ts` | 6 | AI agent runtime |
| `packages/@n8n/instance-ai/src/workspace/daytona-sandbox.ts` | Ongoing | Sandbox lifecycle |
| `packages/frontend/editor-ui/src/features/ai/` | Multiple | AI evaluation wizard |

**Implication for review:** PRs touching `packages/core/` or `packages/cli/`
(core execution path) receive deeper scrutiny and are more likely to need multiple
review rounds.

---

## 10. PR Size and Scope Norms

- **Small, focused PRs** are explicitly required by CONTRIBUTING.md
- **Large PRs** are returned for segmentation
- **Typo-only PRs** are rejected
- **New node PRs** from external contributors are auto-closed unless explicitly
  requested by the n8n team
- **Backport marker** in title: `(backport to release-candidate/X.x)` is appended
  for urgent fixes needing backport

---

## Sources

- 167 merged PRs from `n8n-io/n8n` (PRs #31540–#31859), collected 2026-06-07
- `CONTRIBUTING.md` — Community PR Guidelines section
- `.github/pull_request_template.md` — official PR checklist
- `.github/pull_request_title_conventions.md` — Angular commit message convention
- `ci-check-pr-title.yml`, `ci-pull-requests.yml` — CI gate names
