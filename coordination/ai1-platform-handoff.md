# AI1 (platform) — session resume / onboarding

You are taking over the **AI1 / `ai1-platform`** Claude Code session for the
**Strategic Reports** SaaS, moved from another machine. This file is the durable
context that normally lives in that machine's local memory; read it once, then
read `coordination/ai1-platform.md` (the live log, newest entries at top) for the
current state.

## Who you are
- **AI1 = platform / core-domain.** Dashboard, ServiceM8 integration, webhooks,
  cache, auth, feature gating, and the per-tenant JSON store.
- Peers: **AI2** = phone-bot ("Riley"), **AI3** = customer portal + Stripe billing
  (named "Recover o-AI 3 portal +Stripe" on Dan's screen), plus an **orchestrator**.

## Repos
- **Product code:** `StillframeLLC/servicem8-reports` (private). Next.js 15 App
  Router, React 19, TS strict, Tailwind 4, Railway. Read its `CLAUDE.md` first.
- **Coordination/orchestrator:** `danbh89/strategic-phone-bot`. This repo. Read its
  `CLAUDE.md` + `ORCHESTRATION.md`.
- Clone both on this machine. Paths differ per machine — don't assume the old
  ones. Work the product repo from a clean checkout of `origin/staging`.

## Coordination protocol (do this every task)
1. `git pull` this repo and **read every `coordination/*.md`** before starting.
2. **Write ONLY `coordination/ai1-platform.md`** (newest entry at top). Never edit
   another agent's file — route cross-domain needs through a `→ aiN` note.
3. Shared files are platform/AI1-owned but used by everyone — touch with care and
   flag in the log: `src/lib/servicem8.ts` (esp. `SM8_SCOPES`),
   `src/lib/accounts.ts`, `src/lib/db.ts`, `src/lib/storage.ts`.
4. A change to `SM8_SCOPES` **forces every tenant to sign out + re-OAuth** — flag
   it loudly and verify the scope string against the ServiceM8 API docs first
   (an invalid scope breaks OAuth entirely; this bit us — `read_materials` was
   wrong, the real one is `read_inventory`).

## Pipeline (how every change ships)
- `feat/*` (or `fix/*`/`chore/*`) branch off `origin/staging` → PR vs **staging**.
- Gate before PR: `npx tsc --noEmit`, `npm test`, `npm run build` all green.
  (`npm install` uses `--legacy-peer-deps`.)
- **Auto-merge rule:** after `gh pr create` + green checks, **squash-merge to
  staging immediately** — Dan can't test until it's deployed.
- Dan (or the orchestrator) verifies staging, then **promotes staging→main with a
  MERGE COMMIT (not squash)** + an empty-tree `git diff main staging` check.
- **Do NOT self-promote to main.** Dan/orchestrator signals promotion.
- Commit trailer: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
- Never commit `.env*` or anything under `data/`. Never echo secret values.

### gh-merge gotcha
On the old machine, `staging` was checked out in a git worktree, so
`gh pr merge` **errored locally but still merged remotely** — always verify with
`gh pr view <#> --json state` and `git cat-file -e origin/staging:<file>`. On a
fresh machine you likely won't have that worktree; just branch off
`origin/staging` and merge normally.

## Project facts that aren't obvious from the code
- Storage is a **JSON file store** (`src/lib/db.ts` + `storage.ts`), NOT a DB.
  Per-tenant data under `${DATA_DIR}/users/<accountId>/`. `DATA_DIR` = Railway
  volume `/app/data` in prod.
- **ServiceM8 rotates the refresh token on every use.** ALWAYS get tokens via
  `getValidAccountToken(accountId)` (per-account lock + re-read). Never roll your
  own refresh — concurrent refreshes burn the token (the recurring "401" bug).
- `session.userId` **is the Strategic Reports `account.id`**, not the SM8 UUID.
  Use `findAccountById(session.userId)`.
- Multi-user auth primitives live in `src/lib/tenant-users.ts`
  (`sessionCan` / `hasOwnerAccess` / `adminAccess`) — AI3's domain; don't rework.
- `npm test` parallel-isolation: each test file gets its own temp `DATA_DIR` via
  `vitest.setup.ts` (already in place). Tests should be green in parallel.

## CURRENT STATE (as of the handoff)
- **The Recurring Jobs feature is COMPLETE on `origin/staging`** and Dan has
  approved it. The **orchestrator is promoting staging→main** (~11 commits ahead
  of main at handoff). PRs in the batch: #275, #276, #277, #279, #280, #281, #283,
  #285, #286, #287, #288, #289 — see the `ai1-platform.md` log for details.
- ⚠️ **After promotion, every tenant must sign out + back in once** to re-OAuth
  for the new `read_inventory` scope (the materials-catalog picker 403s until
  then; free-text line items + the rest work regardless). In practice that's just
  Dan reconnecting.
- Promotion also turns on **live SM8 job-writing**: the daily 06:15 cron + the
  "Create now" button create real jobs from active recurring-job templates
  (guards: stale-skip beyond 14 days, max 12/run/template).

## What's next for AI1
1. **After the orchestrator promotes:** sanity-check prod — open `/recurring`, the
   templates panel loads, a "Create now" produces a correct ServiceM8 job — and
   confirm Dan did the re-auth.
2. Deferred / low-priority: a stale-token 401 seen once on an old deploy
   (revisit only if it recurs); optionally set `DATA_DIR=/app/data` explicitly on
   prod+staging (works today via the cwd fallback).
3. **Not AI1's** (leave to AI3): make-Admin permission, portal submit-route email
   config. Don't build these.

When you've onboarded, post a short `STATE` entry to `ai1-platform.md` noting the
session moved to this machine and you're resuming, then `git push`.
