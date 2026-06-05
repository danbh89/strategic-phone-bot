# ai1-platform — coordination log

(Newest entries at top. This file is written ONLY by AI1. I read every other
file in `coordination/` before starting any task.)

### [2026-06-05] STARTING — restricted-user nav polish (Dan staging feedback)
Dan (restricted user) reports the main-dashboard top nav still SHOWS Reports + Recurring (they
redirect, but should be hidden), and `/recurring` redirects to the odd `/advanced/dashboard-config`
landing. These tabs live in `DashboardClient.tsx` (NOT AdvancedNav — those moved to the dashboard
nav), so it's all platform/mine. Branch `feat/restricted-user-nav`. Files (all mine):
- NEW `src/components/usePermissions.ts` (client hook; reads your `/api/account/me` read-only → `can(key)`)
- `DashboardClient.tsx` — hide Reports (×2) + Recurring nav links when `!can(key)`
- `FeatureGate.tsx` — optional `redirectTo` prop (default unchanged)
- `src/app/recurring/page.tsx` — `redirectTo="/dashboard"` so a direct hit bounces to the dashboard.
Security already enforced (#251); this is the nav-hiding UX. → PR vs staging, not self-promoting.

### [2026-06-03] DONE — Phase-3 per-user ACCESS enforcement (product PR #251 → staging)
Merged to staging. The gap Dan found is closed — restricted users no longer reach features
they lack:
- `requireAdvancedFeature(slug)` now also `sessionCan(session,slug)` → 31 advanced API routes 403.
- `/api/account/advanced-features` returns per-user flags (`resolveUserFeatures`) → AdvancedNav
  hides tabs + FeatureGate redirects pages automatically (**no AdvancedNav edit**, as planned).
- Reports core key: `/reports` page redirects + `requirePermission("reports")` on data/export/
  raw-export/schedule. NEW `user-features.ts` + 4 tests. tsc clean · 748 tests · build green.
Owners + legacy-OAuth unaffected. NOT promoted to prod (Dan verifies → go).

**→ ai3 (no action needed):** confirmed — `requireAdvancedFeature` now enforces per-user perms,
incl. your `portal` admin routes (owners/legacy pass). Nav auto-hides via the endpoint. If you
ever want the `company-info`/`manage_users`/`billing` core keys gated the same way on surfaces you
own, shout — I kept to platform surfaces + `reports` here.

### [2026-06-03] STARTING — Phase-3 per-user ACCESS enforcement (Dan found the gap)
Dan's test user (dashboard-only) could still reach reports + every advanced tool — only $ was
masked. #250 gated just the financial routes; the per-user FEATURE/page gating was missing.
Branch `feat/perm-enforcement`. Elegant fix, all platform-owned, **no AI3 file edits**:
- `access.ts requireAdvancedFeature(slug)` → also `&& sessionCan(session, slug)` (covers all 31
  advanced API routes by slug).
- `/api/account/advanced-features` → return per-USER-AND-account flags
  (`resolveFeatures(account)[slug] && sessionCan(session,slug)`). This feeds the hook BOTH your
  AdvancedNav and FeatureGate consume — so nav-hiding + page-redirects fix themselves with **zero
  edits to AdvancedNav.tsx**. NEW `src/lib/user-features.ts` holds the helper (can't live in
  advanced-features.ts — tenant-users.ts imports FEATURE_SLUGS from it → cycle).
- Core keys (not feature slugs): gate the Reports surface on `reports` —
  `src/app/reports/page.tsx` redirect + `requirePermission("reports")` on `/api/sm8/data`,
  `/api/reports/export`, `/api/sm8/raw-export`, `/api/reports/schedule`.

**→ ai3 (FYI, no action):** `requireAdvancedFeature` now ALSO enforces the per-user permission —
any route calling it (incl. your `portal` admin routes) 403s a restricted user lacking that slug.
Owners + legacy-OAuth sessions are unaffected (sessionCan short-circuits). AdvancedNav needs no
change — it hides automatically via the advanced-features endpoint. Shout if you'd rather own the
core-key (`reports`/`company-info`) gating since they neighbor your Admin work. → PR vs staging.

### [2026-06-03] DONE — Phase-3 $-mask (platform half): server-side `view_financials` (product PR #250 → staging)
Merged to staging. Restricted tenant users (no `view_financials`) never RECEIVE $ now:
- **MASK** dashboard `/api/sm8` (summary $ totals, per-job `total`, revenueByMonth → 0; response
  has `financialsMasked:true`) + `/api/sm8/job-materials` (line prices/costs → 0; list still shows).
- **BLOCK** (403, deny-by-default) the financial feeds/exports: `/api/sm8/data`,
  `/api/sm8/recurring-report`, `/api/reports/export`, `/api/sm8/raw-export`.
- NEW `financial-mask.ts` (`canViewFinancials`+`maskDashboardPayload`, pure, 7 tests) +
  `access.requirePermission(key)` built on your `sessionCan`. tsc clean · 744 tests · build green.
Owners + legacy-OAuth sessions unaffected. NOT promoted to prod (held for Dan's verify + go).

**→ ai3 / orchestrator — two fast-follows (not in #250):**
1. **Client-side $-widget hiding in `DashboardClient`** (platform/mine): restricted users currently
   see `$0.00` cards rather than the cards being hidden. The `/api/sm8` response now carries
   `financialsMasked:true` to drive that. I can do it as a follow-up PR — flag if you want it now.
2. **Decision needed:** blocking `/api/sm8/data` also stops restricted users building *non-financial*
   reports (job counts, staff hours). If Dan wants restricted users to build non-$ reports, I'd switch
   `/api/sm8/data` from block → field-level mask instead. → who decides? (Dan / orchestrator)

Verify on staging: as a restricted user (Admin→Users, `view_financials` OFF) the dashboard shows
$0.00 / no job $, job materials show no prices, and report-builder/recurring/export return 403.

### [2026-06-03] STARTING — Phase-3 $-mask (platform half): server-side `view_financials` enforcement
Dan greenlit (his "2"). Branch `feat/financial-mask`. Consuming AI3's
`sessionCan(session,"view_financials")` read-only — thanks for the primitive. Files (all
platform-owned, no open claims):
- NEW `src/lib/financial-mask.ts` (`canViewFinancials` + `maskDashboardPayload`) + test
- `src/lib/access.ts` (+`requirePermission(key)` guard built on `sessionCan`, per your suggestion)
- `src/app/api/sm8/route.ts` — MASK dashboard $ (summary totals, job `.total`, revenueByMonth)
  when restricted; response carries `financialsMasked:true`
- `src/app/api/sm8/job-materials/route.ts` — zero per-line material $ when restricted
- BLOCK (deny-by-default, `requirePermission("view_financials")` → 403) the pure financial
  feeds/exports: `src/app/api/sm8/data`, `/api/sm8/recurring-report`, `/api/reports/export`,
  `/api/sm8/raw-export`.
Scope = server-side enforcement (the real security boundary). FAST-FOLLOW (noted, not this PR):
client-side $-widget HIDING in DashboardClient (server already zeros; `financialsMasked` flag is
ready for it). Blocking `/api/sm8/data` also stops restricted users building *non-financial*
reports — flagging for refinement if Dan wants that allowed. → PR vs staging; NOT self-promoting.

### [2026-06-03] FYI — #247 verified + ON PROD; Admin SM8 controls live; one platform item open
Dan confirmed the staging stack looks good; AI3 promoted it to prod (#249, `064e663`, tree-diff
clean). My **#247** (SM8 reconnect/disconnect endpoint) is **live on prod**; AI3's **#248** wired
the Admin buttons to it — contract matched, no rework. **Company-Info handoff complete** — absorbed
into the Admin tab; the 4 `/api/account/{display-name,logo,timezone,date-format}` routes stayed
platform-owned as agreed. `staging == main` — nothing for me to promote.

**Open platform item (next, on Dan's go):** Phase-3 **dashboard/reports $-mask** — consume AI3's
`sessionCan(session,"view_financials")` to SERVER-SIDE omit/zero $ + job-value fields when the perm
is off, across: dashboard KPIs/$ totals (`/api/sm8/**` + DashboardClient + dashboard-cache), job
cards/detail (`total_invoice_amount`, materials $), reports, recurring MRR/ARR. AI3 owns its own
surfaces (AJS/billing); this is the platform half. Dan said he'll prompt it separately — I'm ready.

### [2026-06-03] DONE — SM8 Reconnect/Disconnect endpoint (product PR #247 → staging)
Merged to staging. Contract is exactly the "→ ai3 CONTRACT" entry below — **AI3 you're
unblocked to wire the Admin buttons:**
- `GET /api/account/sm8-connection` → `{connected:boolean}`
- `DELETE /api/account/sm8-connection` → `{ok:true,connected:false}` (clears account token +
  strips it from the caller's session; no suspend / no `disconnected_at`)
- Reconnect = `window.location.href="/api/auth/login"` (existing OAuth; full nav, not fetch).
Owner-gated via your `hasOwnerAccess`; account-scoped; no `SM8_SCOPES` change. tsc clean,
737 tests, build green. NOT promoted to prod (held for Dan's verify + go-signal). Files: NEW
`src/app/api/account/sm8-connection/route.ts` + `db.deleteToken` + `accounts.clearAccountTokens`
+ a test. Company-Info boundary ack is the entry two below — `company-info/page.tsx` is yours to
relocate; the 4 `/api/account/*` routes stay platform.

### [2026-06-03] STARTING — SM8 Reconnect/Disconnect endpoint (orchestrator GO)
Branch `feat/sm8-reconnect-disconnect`. Files — all platform-owned, no AI3 overlap:
- NEW `src/app/api/account/sm8-connection/route.ts` (GET status + DELETE disconnect)
- `src/lib/db.ts` (+`deleteToken`), `src/lib/accounts.ts` (+`clearAccountTokens`)
- a test under `src/lib/__tests__/`.
Read-only consuming AI3's `hasOwnerAccess` from `tenant-users.ts` for the owner gate (not
editing it — you invited it). Pipeline: → PR vs staging, tsc/test/build green. NOT self-promoting.

### [2026-06-03] → ai3 — SM8 Reconnect/Disconnect endpoint CONTRACT (wire the Admin buttons to these)
Owner-gated (via your `hasOwnerAccess`) + account-scoped (session.userId = account). Reuses
existing OAuth scopes — NO `SM8_SCOPES` change. Landing on staging shortly (PR # in my next DONE).

- **STATUS** — `GET /api/account/sm8-connection`
  → `200 {"connected": boolean}`  (true = a live SM8 token is stored for the account).
  Errors: `401` no session · `403` not owner · `404` no account.
  Use it to pick the button (connected → show "Disconnect"; else → "Reconnect").

- **DISCONNECT** — `DELETE /api/account/sm8-connection`
  → `200 {"ok": true, "connected": false}`.
  Clears the account's stored SM8 token AND strips it from the caller's session (so it can't
  silently refresh back). Deliberately does NOT suspend / set `disconnected_at` — account stays
  active, just no live token. Call: `fetch("/api/account/sm8-connection", { method: "DELETE" })`.

- **RECONNECT** — full browser navigation to `GET /api/auth/login` (EXISTING — do NOT `fetch` it).
  It 302s to ServiceM8 OAuth; the callback restores the token + a fresh session. Wire as
  `window.location.href = "/api/auth/login"`. After it returns, STATUS reads `connected: true`.

  Notes: after DISCONNECT the caller's dashboard (`/api/sm8`) 401s until reconnect — expected.
  Low-risk edge case (internal single-org): another *concurrent* OAuth session for the same
  tenant could repopulate the account token before reconnect (its cookie still holds a copy).
  Fine for now — flag me if you ever want hard global token invalidation.

### [2026-06-03] → ai3 — Company Info ownership boundary (agreed — you're clear to relocate)
Platform-owned, **do NOT edit**: the 4 API routes `/api/account/{display-name,logo,timezone,
date-format}` + their `accounts.ts` data (`display_company_name`/`logo_url`/`timezone`/
`date_format`). They're session-scoped and already work for any logged-in user — just **call**
them from the Admin page. **Handing off to you:** the rendering page
`src/app/advanced/company-info/page.tsx` is pure UI that calls those 4 endpoints — relocate its
JSX into the Admin page and delete the standalone page + its AdvancedNav tab as the orchestrator
directed. If you'd prefer I extract it into a shared component instead, say so; otherwise it's
yours to move. No `accounts.ts` / API-route edits needed from your side.


### [2026-06-03] DONE — job-form config features verified + on prod
- **#239** (emergency-alert SMS + Preferred Appointment toggle) and **#240** (Urgency
  on/off toggle + regenerate-URL passcode): Dan verified both on staging — green. Already
  on prod via **#241** (merge commit, `git diff main staging` clean at promo time).

**→ ai3 / → orchestrator:** heads-up — my #241 staging→main promotion last turn also
carried **#238 (tenant-user Phase-1 foundation) to PROD**, because #238 was on staging
ahead of main when I promoted the whole staging tip (Dan said "push all"). **#242**
(Phase-1 login) wasn't pushed yet, so it stayed staging-only. Net prod state: tenant_users
store + session fields (#238) are live, but NOT the login UI (#242) — additive, backend-
only, unexercised without #242, so should be harmless. Flagging because AI3's file still
lists #238 as staging-only. AI3: confirm that's OK; if you'd rather Phase 1 hit prod as a
unit, no action needed — #242 just promotes on top. Going forward I'll scope promotions to
my own commits, not a blanket staging→main sweep, unless Dan explicitly asks for "all".

**FYI:** Dan has in-flight staging work (currently #242, possibly more) and will promote
to prod himself shortly — so AI1 will NOT initiate any staging→main promotion right now.

**FYI:** AI2's `publish_sms` `SM8_SCOPES` bump + `logGrantedScopes()` in `servicem8.ts` is
reconciled with my STATE — no action. AI2's future "migrate account/admin backup token
racers" (their task #5) will land as a "→ platform" request when they're ready; I'll take it.

---

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
