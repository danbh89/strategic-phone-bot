# ORCHESTRATION.md — how work flows through the agents

This file defines how the orchestrator decomposes a request, dispatches it to the
domain agents, and merges the results back without the agents stepping on each other.

## 1. Roles

- **Orchestrator** (you, the driver): owns decomposition, sequencing, conflict
  resolution, and the merge order. Does not write domain code directly when an agent
  owns it — dispatches instead.
- **Domain agents** (`.claude/agents/*.md`): each owns a slice of
  `StillframeLLC/servicem8-reports` and does the actual editing, testing, and PRs for
  that slice. See the ownership table in [CLAUDE.md](CLAUDE.md).

## 2. Dispatch model

```
request
  → orchestrator decomposes into per-domain tasks
  → for each task: pick the owning agent (by the file-ownership table)
  → agent works on its own feat/* branch off latest staging
  → agent runs tsc + test + build, opens PR vs staging, auto-merges on green
  → orchestrator collects results, resolves cross-domain handoffs
  → Dan verifies staging
  → orchestrator promotes staging → main (merge commit + tree-diff check)
  → Dan verifies prod
```

Independent tasks fan out in parallel. Tasks that share a file (see **Shared files**
in CLAUDE.md) are **serialized** — the owning agent (`platform`) lands its change
first and publishes a handoff; dependent agents rebase onto it.

## 3. Picking the right agent

Route by the file the change lives in, not by the feature's marketing name:

- Touches `src/lib/servicem8.ts`, `sm8-webhooks`, `dashboard-cache`, `session`,
  `access`, `advanced-features`, or `src/app/api/sm8/**` → **platform**
- Touches `phone-bot.ts`, `job-sms.ts`, or Telnyx webhooks → **phone-bot**
- Touches `src/app/portal/**` or `portal-sm8.ts` → **customer-portal**
- Touches `billing.ts`, `src/app/api/billing/**`, or `/upgrade` → **stripe-billing**

If a task spans domains, split it: each agent does its own slice and they coordinate
through a handoff note (never one agent editing another's files).

## 4. The PR pipeline (per the product repo)

```
feat/<scope>  ──PR──▶  staging  ──(Dan verifies)──  PR  ──▶  main  ──(Dan verifies prod)
```

Rules every agent follows:
- Branch from **latest `origin/staging`**, never from `main` or a stale base.
- Gate every PR on `npx tsc --noEmit`, `npm test`, `npm run build`.
- **Auto-merge to staging**: after `gh pr create` + green tests, squash-merge
  immediately — Dan can't test a change until it's deployed to staging.
- **Promotions to `main` use merge commits**, then verify with
  `git diff main staging` (empty tree diff = nothing dropped). Squash promotions have
  silently diverged `main`/`staging` and dropped work before — do not squash to main.
- One logical change per commit; commit message in imperative mood with a *why* body.

## 5. Coordination state (lives in the product repo)

- `SESSION_LOG.md` — append-only event log. Every agent reads it before starting and
  appends on every commit/merge so parallel agents see each other's moves.
- `PARALLEL_SESSION_HANDOFF.md` — the authoritative file-ownership boundaries and the
  shared-file rules. When in doubt about ownership, this wins.
- Handoffs between agents in *this* orchestrator are dropped under
  [`handoff/`](handoff/README.md).

## 6. Known coordination hazards (learned the hard way)

- **SM8 scope changes** (`SM8_SCOPES` in `servicem8.ts`) force **every tenant to
  re-authorize**. Only `platform` changes it, and only after flagging it. Update the
  scope-count assertion in `src/lib/__tests__/servicem8.test.ts` in the same change.
- **SM8 rotates refresh tokens on every use.** Never roll a bespoke token refresh —
  always go through the shared `getValidAccountToken` (per-account lock + re-read).
  Concurrent refreshes for one tenant race and 401.
- **Squash-merge divergence**: a GitHub squash-merge of a staging PR diverges
  `main`/`staging` and the next promotion can silently drop a file. Always promote
  with merge commits + the tree-diff check.
- **Checkout topology**: the platform clone may sit on a detached HEAD off
  `origin/staging` while a separate worktree holds the `staging` branch checked out.
  `gh pr merge` can error locally (worktree holds the branch) yet still merge on the
  remote — verify with `gh pr view <n> --json state` rather than trusting the CLI exit.

## 7. Definition of done for a dispatched task

1. Change is on `staging`, green (tsc + test + build).
2. A handoff note exists if another domain must react.
3. `SESSION_LOG.md` updated.
4. The plain-English "what changed / how to verify on staging" is reported back to Dan.
