# CLAUDE.md — Strategic Phone Bot Orchestrator

## What this repo is
This is the **orchestrator workspace** for the Strategic Reports ecosystem. It does
**not** hold application code. It holds the coordination process and the subagent
definitions used to drive parallel work across the product's four domains.

The actual product code lives in **`StillframeLLC/servicem8-reports`** — a single
Next.js 15 (App Router) monolith deployed on Railway. Historically that codebase was
worked by parallel Claude sessions (AI1 = platform, AI2 = phone bot, AI3 = customer
portal + Stripe billing). This orchestrator formalizes that split into four named
subagents so one driver can fan work out and merge it back safely.

- **Product repo:** `StillframeLLC/servicem8-reports` (private)
- **Orchestrator repo:** `danbh89/strategic-phone-bot` (this repo)
- **Working branch convention:** `claude/<slug>` (e.g. this branch)

## The four domains
Each maps to an agent in `.claude/agents/` and to an ownership area in the product repo.

| Agent | Domain | Primary code (in servicem8-reports) |
|-------|--------|--------------------------------------|
| `platform` | Core app: dashboard, SM8 integration, webhooks, cache, auth, feature gating | `src/components/DashboardClient.tsx`, `src/components/dashboard/`, `src/lib/servicem8.ts`, `src/lib/sm8-webhooks.ts`, `src/lib/dashboard-cache.ts`, `src/lib/session.ts`, `src/lib/access.ts`, `src/lib/advanced-features.ts`, `src/app/api/sm8/**` |
| `phone-bot` | Telnyx/TexML voice agent "Riley": inbound calls → job creation, SMS | `src/lib/phone-bot.ts`, `src/lib/job-sms.ts`, `src/app/api/telnyx/**` |
| `customer-portal` | Customer-facing portal: email/magic-link auth, site selection, job views | `src/app/portal/**`, `src/lib/portal-sm8.ts`, `src/app/api/portal/**` |
| `stripe-billing` | Subscriptions, trials, plans, checkout, billing webhooks | `src/lib/billing.ts`, `src/app/api/billing/**`, `src/app/upgrade/**` |

## Shared files (touch only with coordination)
These are owned by `platform` but used by everyone. A change here can break any domain,
so a non-platform agent must flag it in a handoff rather than edit unilaterally:
- `src/lib/servicem8.ts` (esp. `SM8_SCOPES` — scope changes force every tenant to re-auth)
- `src/lib/accounts.ts` / `src/lib/db.ts` (the JSON account + token store)
- `src/lib/storage.ts`

## How to drive this
Read **[ORCHESTRATION.md](ORCHESTRATION.md)** for the full pipeline and dispatch model,
and **[handoff/README.md](handoff/README.md)** for the cross-agent handoff protocol.
The short version:

1. The orchestrator decomposes a request and dispatches each piece to the agent that
   owns those files.
2. Every code change follows the product repo's PR pipeline:
   `feat/*` branch → PR vs `staging` → green CI → auto-merge to staging → Dan verifies
   staging → PR `staging` → `main` → Dan verifies prod.
3. Before any PR: `npx tsc --noEmit`, `npm test`, `npm run build` must pass.
4. Promotions to `main` use **merge commits** + a `git diff main staging` tree check —
   squash promotions have silently dropped work before.

## Non-negotiables (inherited from the product repo)
- Never commit `.env*` or anything under `data/`. Never echo secret values.
- Installs use `npm install --legacy-peer-deps`.
- `main` is production (Railway auto-deploys within minutes). Always branch.
- Dan can't review diffs — his real check is **staging behaving correctly**. Describe
  the plan in plain English first; don't block on post-hoc diff review.
- End commit messages with:
  `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`
