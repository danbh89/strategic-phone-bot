# ai1-platform — coordination log

(Newest entries at top. This file is written ONLY by AI1. I read every other
file in `coordination/` before starting any task.)

### [2026-06-03] AI1 online — protocol adopted
Standing session. Cloned `../coord`, read `coordination/` (no peer files present yet —
I'm first in). Will pull + read all files before each task, append newest-at-top to this
file only, claim files with a STARTING entry before editing, and route shared-file
(`servicem8.ts`/`accounts.ts`/`db.ts`/`storage.ts`) requests from other AIs through a
"→ platform" note. Continuing current platform work under these rules.

---

### [2026-06-03] STATE — AI1 (platform)

**Domain + files I work in**
- Core app / platform. Product repo: `StillframeLLC/servicem8-reports` (Next.js 15,
  App Router, Railway, JSON file store — no DB).
- Owned areas: dashboard (`src/components/DashboardClient.tsx`, `src/components/dashboard/`),
  SM8 integration (`src/lib/servicem8.ts`, `src/lib/sm8-webhooks.ts`,
  `src/lib/dashboard-cache.ts`, `src/app/api/sm8/**`), auth (`src/lib/session.ts`,
  `src/lib/access.ts`), feature gating (`src/lib/advanced-features.ts`),
  job-form config plumbing (`src/lib/job-types-config.ts`, `src/lib/job-sms.ts`,
  `src/app/advanced/customer-forms/`, the three submit routes).
- **Shared files I own (others request changes via "→ platform"):**
  `src/lib/servicem8.ts`, `src/lib/accounts.ts`, `src/lib/db.ts`, `src/lib/storage.ts`.

**Done (recently shipped — all on prod/main as of today)**
- #224 `sm8FetchAll` pagination-loop fix (SM8 $top/$skip could loop → cache built from
  ~700 of 20k dupes; now dedupes during pagination + stops on an all-dupe page).
- #227 `getValidAccountToken` — single per-account-serialized token refresh helper
  (SM8 rotates refresh tokens on every use; concurrent refreshes raced → 401).
- #228 scheduler routed through that helper (clock-in/notification/sweep crons were
  the real recurring-401 source).
- #231 dashboard freshness (reload self-heals via forced-incremental `sync=1`) +
  recent-jobs now sorted by `edit_date` not scheduled `date`.
- #239 emergency-alert SMS (fires when a Request Type label contains "emergency") +
  Preferred Appointment on/off toggle + default "Emergency Call (Same Day)" request type.
- #240 Urgency section on/off toggle + Advanced-Tools passcode re-prompt to regenerate
  the public job-form URL.
- #241 promoted all of the above (plus #238) staging → main (merge commit, tree-diff verified).

**In progress**
- None active. Standing by for orchestrator/Dan direction.

**Not started / backlog**
- Urgency DEFAULT change app-wide (ASAP 1-3d / Less Urgent 2-5d) — Dan floated it, never
  confirmed; left code defaults untouched. Needs a yes/no.
- Optional live dashboard auto-refresh (server→client push/poll) — discussed, not scoped.
- Materials-only edits: unconfirmed whether SM8 fires `job.updated` when only line
  items/materials change (affects real-time job value). May need a JobMaterial webhook.

**Open bugs / known issues**
- Recurring token-401 should be resolved by #227/#228 — still want to confirm tenants
  `f0d57a10` / `f13d94c2` stopped erroring in Railway logs (genuinely-revoked tenants now
  get flagged disconnected instead of retried forever).
- ESLint backlog; `next build` is NOT gated on lint (`eslint.ignoreDuringBuilds: true`).

**Integrations I rely on (env var NAMES only — no secrets)**
- ServiceM8 OAuth2: `SERVICEM8_APP_ID`, `SERVICEM8_APP_SECRET`. Scope list in
  `SM8_SCOPES` (servicem8.ts) — currently includes `publish_sms`/`publish_email`. SM8
  REST base + `SM8_PLATFORM_BASE` (platform_service_email/_sms, webhook_subscriptions).
  Scope changes force every tenant to re-auth.
- Railway: deploys `main` automatically. `DATA_DIR` (volume) backs the JSON store;
  `NEXTAUTH_SECRET` derives the AES-256-GCM session-cookie key. Staging service
  `servicem8-reports-staging`, project `b256f579-9420-44f4-94c8-7fb05b54b965`, env `production`.
- Email: `RESEND_API_KEY` (Resend) + SM8 `/platform_service_email`. SMS via SM8
  `/platform_service_sms`.
- Maps: `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` (HTTP-referrer restricted).

**FYI for other AIs**
- #238 (tenant-user model + permissions foundation) touched shared files
  (`session.ts`, `accounts.ts`, `db.ts`, new `tenant-users.ts`/`password.ts`). It's
  merged + on prod, backend-only (no UI). Going forward those shared files are
  platform/AI1 — send a "→ platform" note for changes there.

**Next 3-5 tasks (priority order)**
1. Stand by for orchestrator/Dan direction; keep this file current per protocol.
2. Verify recurring token-401 cleared on prod via Railway logs.
3. Smoke-check #239/#240 on prod after deploy (toggles hide sections; emergency SMS fires;
   regenerate prompts for the passcode).
4. (Backlog) get Dan's call on the app-wide urgency default change.
5. (Backlog) scope optional live dashboard auto-refresh if Dan wants it.
