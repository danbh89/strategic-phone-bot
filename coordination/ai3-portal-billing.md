# ai3-portal-billing — coordination log

(Newest entries at top. This file is written ONLY by AI3. I read every other
file in `coordination/` before starting any task.)

### [2026-06-08] STARTING — my half of the OAuth-bypass batch (sequencing w/ AI1 feat/oauth-bypass-hardening)
AI1 started the callback hardening + asked for my two bits in the SAME staging batch. Doing them now.
Branch `feat/login-bypass-ai3-half`. Files (mine): `src/app/login/page.tsx` (remove the "Connect with
ServiceM8" button/block — existing users sign in with password; new tenants still enter via home/pricing
→ AI1's callback keeps the no-owner bootstrap) + `src/lib/tenant-users.ts` (tighten `hasOwnerAccess` &
`sessionCan`: a legacy OAuth session (no tenantUserId) is full-access ONLY when `accountHasOwner(userId)`
is false; once an owner exists it's NOT privileged). Composes with AI1's callback (defense-in-depth) and
with adminAccess (make-admin, queued next). NOT self-promoting. Will post DONE + ping you when mine is on
staging; promote together after Dan verifies (password login works · owner Reconnect stays in · fresh
OAuth on owner-tenant → /login). FYI optional follow-up for you: the home-page "Sign In" link still →
`/api/auth/login`; could point it at `/login` to skip a pointless OAuth round-trip for existing users
(your file, not touching it).

### [2026-06-08] ACCEPTED + QUEUED — "Admin" as a per-user permission (make-admin), AFTER this batch
Dan approved. Reuse `manage_users` as the Admin grant (key confirmed). adminAccess(session) =
hasOwnerAccess || sessionCan(manage_users); swap Admin gates (tab/page/users routes) to adminAccess;
relabel the checkbox; surface adminAccess on /api/account/me. GUARDRAILS (Dan-approved): only the OWNER
may grant/revoke `manage_users` + transfer ownership; **only the owner manages other admins** (admins
manage non-admin users); owner row non-removable. Sequenced right AFTER the bypass batch lands (same
legacy-rule area). Will open a separate PR.

### [2026-06-05] ack platform #255 — admin key IS `manage_users`
Confirmed: my admin/owner permission key is `manage_users` (in PERMISSION_KEYS). Your
`/api/account/sm8-connection` "owner OR sessionCan('manage_users')" gate matches — no change
needed. Thanks for the forward-compat.

### [2026-06-05] → platform — HARDEN THE OAUTH LOGIN BYPASS (Dan-flagged security issue)
Dan: ServiceM8 OAuth currently logs ANYONE who can authorize the add-on straight in with
full access — it's a backdoor around the new password login. The `/api/auth/login` link sits
on ~6 pages (login, home ×3, pricing, community, dashboard reconnect). Removing buttons is
cosmetic; the real fix is the **OAuth callback** (`src/app/api/auth/callback/servicem8/route.ts`
— your file). Requested behavior (Dan-confirmed intent):
- If the callback's account **has an owner login** (`accountHasOwner(account.id)` from
  `tenant-users.ts`): do NOT mint a full-access app session for the authorizer. Refresh the
  SM8 token as today, then **redirect to `/login`** — UNLESS a valid tenant-user (password)
  session already exists (the owner's Admin "Reconnect"), in which case preserve it + return
  to /admin or /dashboard.
- If the account has **no owner yet**: keep today's bootstrap behavior (OAuth → full session →
  onboarding/create-login) so brand-new tenants can still get in.
- New-tenant signup entry on the marketing/pricing pages stays (those create the account).
Dan has owner logins on staging+prod for his tenant; his 2nd tenant has no owner → bootstrap
path keeps it working. This is the linchpin that closes the bypass for ALL entry points.
My lane lands in concert: remove/replace the `/login` "Connect with ServiceM8" button + tighten
the `hasOwnerAccess`/`sessionCan` legacy rule to "OAuth-session full access ONLY when the
account has no owner." Sequence: your callback change + my bits ship together to staging, Dan
verifies he can still log in (password) + Reconnect works, THEN prod. Flagging timing so we
don't lock Dan out.

### [2026-06-05] DONE — transfer ownership to another user (product PR #256 → staging)
Merged to staging (6d7e52b). Admin → Users "Make owner" on an active user → `transferOwnership`
(promotes target to owner+all perms, demotes old owner to regular user keeping perms, single-owner
invariant). `POST /api/account/users/[id]/make-owner` (owner-gated, account-scoped) re-syncs the
caller's session. tsc clean, 751 tests, build exit 0. Staging-only. No platform/shared files.
Still open: the OAuth-bypass hardening (→ platform request above; AI1's callback) + the
dashboard/reports $-mask (platform).

### [2026-06-05] STARTING — transfer ownership to another user (AI3 lane)
Dan wants an owner to hand owner/admin status to a different user. Branch
`feat/transfer-ownership`. Files (mine): `src/lib/tenant-users.ts` (+`transferOwnership` with
single-owner invariant + tests), `src/app/api/account/users/[id]/route.ts` (or new
make-owner action) — owner-gated, target must be an ACTIVE user of the same account; new user
→ is_owner=true (all perms), old owner → is_owner=false but KEEPS all perms (least-surprising;
new owner can adjust); if the acting session is the old owner, downgrade their session. Admin
Users UI: "Make owner" action on active users with a strong confirm. No platform/shared files.
PR → staging.

### [2026-06-03] DONE — PROMOTED staging→main (multi-user auth + Admin tab + SM8 endpoint) — ON PROD
Merge commit `064e663`; `git diff main..staging` empty afterward (in sync). Now LIVE on prod:
#242 login, #243 logout/UX, #244 user mgmt, #245 billing enforcement + `sessionCan`, #246 Admin
tab, #247 (platform SM8 reconnect/disconnect endpoint), #248 SM8 wire-up. @platform: your #247 is
now on prod. Remaining open: dashboard/reports $-mask (platform; Dan handling that prompt). Owners
create their login via `/account/create-login`; existing tenants keep using OAuth until they do.

### [2026-06-03] → orchestrator + platform — PROMOTING staging→main now (Dan's direct go)
Dan verified the staging stack and told me (AI3) directly to push it to prod. Promoting the
whole staging tip → main via MERGE commit + `git diff main staging` tree check. Contents (clean —
this feature only, no unrelated/other-session commits): #242 Phase-1 login · #243 logout/create-login
UX · #244 Phase-2 user mgmt · #245 billing enforcement · #246 Admin tab · **#247 (platform's
SM8 reconnect/disconnect endpoint — heads-up @platform, your #247 goes to PROD with this)** · #248
SM8 wire-up. I know promotions usually route through you/AI1 — doing it here per Dan's explicit
instruction. Will post DONE with the merge commit. NOTE: dashboard/reports $-mask is NOT in this
stack (still open with platform); Dan said he'll handle that prompt separately.

### [2026-06-03] DONE — wired Admin ServiceM8 Disconnect to AI1's endpoint (product PR #248 → staging)
AI1 shipped #247 (sm8-connection endpoint), so the Admin tab's SM8 controls are now fully live.
Rewired `Sm8ConnectionControls`: GET `/api/account/sm8-connection` → Connected/Not-connected
badge + button choice; Disconnect now `DELETE /api/account/sm8-connection` (was the stubbed path);
Reconnect unchanged (`/api/auth/login`). Client-only. tsc clean, 737 tests, build exit 0.
Admin tab (reconnect/disconnect + Company Info + Users) is now feature-complete on staging.
Thanks @platform — your #247 contract matched exactly. Remaining open: Phase-3 dashboard/reports
$-mask (still with platform — consume my `sessionCan(session,"view_financials")`). Staging-only.

### [2026-06-03] DONE — Admin tab consolidation (product PR #246 → staging)
Merged to staging (7d52107). Owner-only **Admin** tab (`/advanced/admin`, renamed from
"Users"): hosts ServiceM8 connection controls + Company Info (relocated rendering) + Users
mgmt. Company Info standalone tab REMOVED → admin-only; `/advanced/company-info` +
`/advanced/users` redirect to `/advanced/admin`. Reconnect = `/api/auth/login` (live).
Disconnect calls `POST /api/account/sm8-disconnect` (STILL PENDING from platform — button
reports "not available yet" until it lands). No platform files touched (Company Info was a
self-contained client component). tsc clean, 733 tests, build exit 0. Awaiting Dan staging
verify. Staging-only.
→ platform: (1) my `POST /api/account/sm8-disconnect` request (clear token only, no suspend)
is the one remaining dep to make Disconnect live. (2) FYI `advanced/page.tsx` landing +
a `DashboardClient` link still point at `/advanced/company-info` (work via redirect; you may
want to hide/repoint the company-info entry for non-owners — your files, I didn't touch them).
Still open from before: the Phase-3 dashboard/reports $-mask (consume my `sessionCan`).

### [2026-06-03] STARTING — Admin tab consolidation (orchestrator dispatch) + → platform (Disconnect endpoint)
Executing the Admin-tab dispatch. Branch `feat/advanced-admin-tab`.
**Boundary finding (Company Info):** `/advanced/company-info/page.tsx` is a SELF-CONTAINED
client component — it reads/writes ONLY via fetch to existing platform APIs
(`/api/account/display-name|logo|date-format|timezone`) and imports no platform-owned
component (just the pure `@/lib/timezone-options` helper, read-only). So I'm relocating the
RENDERING into the Admin page and NOT editing `accounts.ts` or any `/api/account/*` route.
→ platform: flag if you consider the company-info *page file* yours; I read it as shared
/advanced UI in my dispatch. I touch zero platform data code.
**Files (mine/new):** new `src/app/advanced/admin/page.tsx` (owner-gated hub), move
`UsersClient.tsx` into it, new `src/components/advanced/CompanyInfoPanel.tsx` (relocated
rendering), new `src/components/advanced/Sm8ConnectionControls.tsx`; redirect stubs at
`/advanced/users` + `/advanced/company-info` → `/advanced/admin`.
**SHARED (claiming):** `src/components/advanced/AdvancedNav.tsx` — rename Users→Admin
(href `/advanced/admin`, owner-gated as today), REMOVE the Company Info tab.
**Reconnect:** wired to existing `/api/auth/login` (re-run OAuth) — no platform dep.
**→ platform — Disconnect endpoint I need:** `POST /api/account/sm8-disconnect` — clears the
SESSION account's stored SM8 token only; MUST NOT `markAccountDisconnected` / set suspended
(user-initiated); no `SM8_SCOPES` change; return `{ ok: true }`. Until it lands my Disconnect
button calls that path and reports "not available yet" gracefully (stubbed per Dan).
PR → staging; tsc+test+build green; no prod promote (Dan verifies + signals).

### [2026-06-03] DONE — Phase 3 AI3 slice: sessionCan + billing enforcement (product PR #245 → staging)
Merged to staging (36c0e38). `sessionCan(session,key)` added to `tenant-users.ts` (owner/legacy
→ true; else stored perm) — **platform: this is the primitive for your requirePermission +
view_financials $-mask, please consume it.** Enforced `billing` perm on `/upgrade` +
checkout-session + portal-session (owner/legacy bypass). tsc clean, 733 tests, build exit 0.
The dashboard/reports $-mask remains with platform (handoff below). Admin-tab still held on AI1's
reconnect/disconnect endpoint + Company-Info boundary. All staging-only.

### [2026-06-03] ack + → platform — #238-on-prod OK; Admin-tab accepted (seq'd); Phase-3 $-mask handoff
**ack #238 partial-prod:** Fine — #238 (tenant_users store + session fields) on prod is
additive/backend-only and inert without #242. No action; #242/#244 promote on top when Dan
gives the go. I'll stop calling Phase 1 "staging-only".

**ack Admin-tab directive (orchestrator):** Accepted. I own the rename Users→Admin + hosting
the SM8 reconnect/disconnect UI + absorbing Company Info into the Admin page. Respecting the
orchestrator sequencing — I will NOT wire it until (a) AI1's reconnect/disconnect endpoint
exists and (b) the Company-Info ownership boundary is agreed below. Holding the UI work.

**→ platform (Company Info boundary, for the Admin tab):** Please confirm what under
"Company Info" (`/advanced/company-info`) is yours. My read: the page composes platform data
(`accounts.ts` display-name/logo/timezone/date-format helpers + their API routes). Proposed
boundary: I move the *rendering* into the Admin page and call your existing endpoints/helpers;
I do NOT edit `accounts.ts` or platform data routes. If any Company-Info *component* is yours,
hand it off or tell me it's fine to relocate the JSX. I'll wait for your ack before touching it.

**→ platform (Phase 3 $-mask + route gating — the big one):** Dan greenlit Phase 3. The
permission model is mine; the dollar/job-value data is yours. Contract:
- I'm adding `sessionCan(session, key)` to `src/lib/tenant-users.ts` (owner OR legacy-OAuth
  session [no tenantUserId] → true; else `session.permissions[key] === true`). Single source
  of truth — please consume it, don't reimplement.
- Money gate key = `view_financials`. When `!sessionCan(session,"view_financials")`, the
  SERVER must omit/zero $ + job-value fields before sending (real enforcement, not just CSS).
  Surfaces in your lane: dashboard KPIs/$ totals (DashboardClient + `/api/sm8/**` +
  dashboard-cache), job cards/detail (`total_invoice_amount`, materials $), reports, recurring
  MRR/ARR. I'll handle my own surfaces (AJS, billing).
- Suggest a `requirePermission(key)` guard in `access.ts` (your file) built on `sessionCan`,
  mapping advanced slugs → permission keys, for per-user page/route gating.
This is sequenced platform-first per the orchestrator; I'm shipping the primitive + my-lane
enforcement now so you have `sessionCan` to build on.

### [2026-06-03] STARTING — Phase 3 (AI3 slice): sessionCan primitive + gate billing surfaces
Branch `feat/perm-enforcement-billing`. Files (all mine): `src/lib/tenant-users.ts`
(`sessionCan` + spec), `src/app/upgrade/page.tsx` + `src/app/api/billing/checkout-session/route.ts`
+ `src/app/api/billing/portal-session/route.ts` (require `billing` permission; owner/legacy bypass).
No shared/platform files. → PR staging.

### [2026-06-03] DONE — Phase 2: tenant user management (product PR #244 → staging)
Owner-only Users area in Advanced Tools merged to staging (3e43fce). `/advanced/users`
(invite + per-feature permission checkboxes + edit/disable/remove; owner row protected),
APIs `GET /api/account/me`, `GET/POST /api/account/users`, `PATCH/DELETE /api/account/users/[id]`,
`POST /api/auth/set-password` (all owner-gated + account-scoped), `/set-password` page, Resend
invite email + copy-paste link fallback, owner-only "Users" tab in AdvancedNav (the only shared
file touched). Permissions are STORED, not yet enforced — that's Phase 3. tsc clean, 731 tests,
build exit 0. Awaiting Dan staging verify. Still NOT promoted to prod (Phases 1+2 all staging-only).
Reminder: my `→ platform` SM8 Reconnect/Disconnect request (below) is still open for AI1.

### [2026-06-03] → platform — please add a tenant-facing ServiceM8 Reconnect/Disconnect control in Advanced Tools
Dan wants, in `/advanced`, a "Reconnect ServiceM8" (re-run OAuth to refresh the token) and/or
"Disconnect then reconnect" button. This is your domain — SM8 OAuth flow (`/api/auth/login`),
the account token store (`accounts.ts`/`db.ts`), and your auto-suspend-on-disconnect /
`markAccountDisconnected` logic all need to agree. Reconnect ≈ surface `/api/auth/login`;
disconnect ≈ clear the stored token (and presumably NOT trip a "suspended" state since it's
user-initiated). I did NOT build it (stayed out of your token/auth code). If you'd rather I
host the button on a page I own and just call an endpoint you expose, say so and I'll wire the UI.

### [2026-06-03] STARTING — Phase 2: tenant user management UI (owner-gated) in Advanced Tools
Building the self-serve user-management feature Dan asked for: owner invites additional users
with specific per-feature permissions. All AI3 lane. NEW files: `src/app/advanced/users/` (page +
client), `src/app/set-password/` (page + client), `src/app/api/account/me/route.ts`,
`src/app/api/account/users/route.ts` (+`[id]/route.ts`), `src/app/api/auth/set-password/route.ts`,
`src/lib/tenant-user-email.ts`, `src/components/advanced/useOwnerAccess.ts`; extend my
`src/lib/tenant-users.ts`. SHARED edit (claiming — no open claim): `src/components/advanced/AdvancedNav.tsx`
— add an owner-only "Users" tab (gated by my new /api/account/me, not advanced-features). Owner-gate
= session.isOwner OR a legacy OAuth session (no tenantUserId) treated as full-access, consistent with
Phase 1. Invite flow: create invited user → emailed set-password link (Resend) + show the link in the
UI as a copy-paste fallback. Enforcement of the granted permissions across the app stays Phase 3.
Branch `feat/tenant-user-management` → PR staging.

### [2026-06-03] DONE — logout→/login + create-login form UX (product PR #243 → staging)
Both merged to staging (db28737). (1) `/api/auth/logout` redirects to `/login` (was `/`) —
still only clears the session cookie, SM8 token untouched, so sign-in reuses it (no
re-OAuth). (2) `CreateLoginClient` validates live (green password hint, mismatch hint,
submit disabled until valid) — no more field-wipe on a bad password. Did NOT touch
`DashboardClient` (Sign Out button href unchanged) or any shared/platform file. tsc clean,
727 tests, build exit 0. Awaiting Dan's staging verify. Phase 1 (multi-user login) also
still staging-only — not promoted to prod yet.

### [2026-06-03] STARTING — logout→/login + create-login form UX (Dan, post Phase-1 staging verify)
Dan verified Phase 1 login on staging (all 4 steps ✓). Two small fixes:
1. **Sign-out → `/login`** instead of `/`. NOTE: logout never disconnected ServiceM8 — it
   only clears the session cookie; the SM8 token persists on the account, so password
   sign-in reuses it (no re-OAuth). Pure redirect-target change. The "Sign Out" button in
   `DashboardClient` is unchanged (href stays `/api/auth/logout`), so I'm NOT touching that file.
2. **Create-login form "wipe"**: live password validation + disable submit until valid;
   never clears typed fields (likely an autofill+re-render reset on failed submit).
Files I'll touch: `src/app/api/auth/logout/route.ts` (auth route, no existing claim),
`src/app/account/create-login/CreateLoginClient.tsx` (mine). Branch `feat/auth-logout-and-create-login-ux` → PR staging.
→ platform (FYI, not a request): I'm editing `src/app/api/auth/logout/route.ts` — a 1-line
redirect (`/` → `/login`) as part of the AI3 login feature. It's not in your owned-files
list and has no open claim; shout if you'd rather own it.

### [2026-06-03] AI3 online — protocol adopted
Standing session. Pulled `../coord`, read `coordination/ai1-platform.md` (AI2 file not
present yet). Adopting the protocol: pull + read all coordination files before each task;
append newest-at-top to THIS file only; claim files with a STARTING entry before editing;
keep code in the product repo on `feat/*` → PR → staging. Shared/platform files
(`servicem8.ts`, `accounts.ts`, `db.ts`, `storage.ts`, `session.ts`, `access.ts`,
`advanced-features.ts`) belong to AI1 — I'll send a "→ platform" note for changes there,
not edit them myself. Standing by.

→ platform (FYI, no action): my merged Phase-1 auth work (#238) already added a
`tenant_users` collection + `TenantUserRow` to `db.ts` and optional `tenantUser*` fields
to `session.ts`. That's done/on prod. From here I won't touch those shared files without a
"→ platform" request — but Phase 3 (below) will likely need one (server-side $-masking in
shared data paths).

---

### [2026-06-03] STATE — AI3 (customer portal + Stripe billing + multi-user auth)

**Domains + files I work in**
- **Stripe billing:** `src/lib/billing.ts`, `src/app/api/billing/**` (checkout-session,
  portal-session, webhook), `src/app/upgrade/**` (page + UpgradeClient), the five
  `stripe_*` AccountRow fields.
- **Customer portal:** `src/app/portal/**`, `src/app/api/portal/**`, `src/lib/portal-*.ts`
  (portal, portal-session, portal-magic-link, portal-sm8, portal-audit),
  `src/components/portal/**`.
- **Multi-user auth (new, AI3 lane):** `src/lib/tenant-users.ts`, `src/lib/password.ts`,
  `src/app/api/auth/password-login/`, `src/app/login/`,
  `src/app/api/account/create-owner-login/`, `src/app/account/create-login/`.
  (Uses shared `db.ts`/`session.ts`/`advanced-features.ts` read-only or via already-merged
  additions — further changes there go through "→ platform".)
- **Active Jobs Summary (legacy AI3):** `src/lib/active-jobs-summary.ts` + its API routes +
  `src/components/advanced/ActiveJobsSummary.tsx`.

**Done (on PROD/main)**
- Stripe foundation + Checkout (M8 #140), webhook + Customer Portal + reconciliation
  (M9 #141), live-mode cutover (#170, verified via `VERIFY2026` coupon).
- Stripe Checkout trial synced to the in-app trial (#220): `trial_period_days` = whole
  days left on `trial_ends_at`, omitted when 0 (no double free month on /upgrade).

**Done (on STAGING, awaiting Dan verify — NOT yet promoted to prod)**
- Multi-user auth **Phase 1**: foundation (#238 — tenant_users store, scrypt
  `password.ts`, `tenant-users.ts` permissions/owner/invite logic, session fields) +
  login (#242 — `POST /api/auth/password-login`, `/login`, `POST
  /api/account/create-owner-login` + `/account/create-login` owner bootstrap). Additive:
  SM8 OAuth login still works; enforcement deferred to Phase 3.

**In progress**
- None active. Holding for Dan to verify Phase 1 login flow on staging.

**Not started**
- **Phase 2** — self-serve user-management UI: owner-gated Users page; invite via Resend +
  `/set-password`; per-feature permission editor; `POST/GET/PATCH/DELETE
  /api/account/users/**`. Stays in AI3 lane (new files + `tenant-users.ts`).
- **Phase 3** — enforcement: `requirePermission` on routes/pages + server-side stripping
  of $/job-value fields when `view_financials` is off + client nav/section hiding. WILL
  need "→ platform" coordination (shared data paths / `access.ts` / dashboard).
- (Backlog) `successUrl`/`cancelUrl` override on checkout-session so the onboarding
  "Subscribe" flow lands on `/dashboard` not `/upgrade` — AI1 requested twice.

**Open bugs / known issues**
- Phase 1 not yet verified on staging by Dan; not on prod.
- Lib tests bind `DATA_DIR` at import, so account/tenant-user specs write to the real
  local `./data` (gitignored) — use unique ids per run (done). Harmless, not prod.

**Integrations I rely on (env var NAMES + non-secret IDs only)**
- **Stripe** account `acct_1TCgJ7L3CTpMN6l0` (SM8 Pay-managed).
  - LIVE prices: Basic `price_1TcsEzL3CTpMN6l0n4YrZhua` ($29), Pro
    `price_1TcsF8L3CTpMN6l0q3l66OHI` ($79), Enterprise `price_1TcsFCL3CTpMN6l02T0CsP6u`
    ($149). TEST prices: Basic `price_1TcEvXL3CTpMN6l0AAYUzNIX`, Pro
    `price_1TcEvYL3CTpMN6l0GzEoNugL`, Enterprise `price_1TcEvZL3CTpMN6l0WHlmg7Wz`.
  - Verification coupon `VERIFY2026` (id `Yk7q9KrR`, 100% off, max 1).
  - Webhooks: prod `strategicreporting.tech/api/billing/webhook`, staging
    `mindful-enchantment-production-4f66.up.railway.app/api/billing/webhook` (events:
    customer.subscription.{created,updated,deleted}, invoice.payment_failed).
  - Env NAMES: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_PRICE_ID_BASIC`,
    `STRIPE_PRICE_ID_PRO`, `STRIPE_PRICE_ID_ENTERPRISE` (staging = test keys, prod = live).
- **ServiceM8**: portal uses tenant OAuth token (via AI1's `getValidAccountToken`);
  portal scopes incl. `manage_customer_contacts`. `read_security_roles` is available but
  unused (we chose SR-native auth over SM8 role gating).
- **Email**: `RESEND_API_KEY` (Resend) — will back the Phase-2 invite emails.
- **Passwords**: scrypt via Node `crypto` — NO new dependency, no env var.
- **Railway**: prod `servicem8-reports` (strategicreporting.tech), staging
  `servicem8-reports-staging`.

**Next 3-5 tasks (priority order)**
1. Hold for Dan's staging verification of Phase 1 login + owner bootstrap.
2. Phase 2 — user-management UI (invite/permissions/disable/remove). AI3 lane.
3. Phase 3 — enforcement + server-side $-mask. Will open a "→ platform" request for the
   shared data-path changes before starting.
4. Promote the multi-user feature staging → main once Phases 1–3 verified.
5. (Backlog) checkout-session `successUrl` override for the onboarding plan-picker.
