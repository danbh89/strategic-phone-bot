# ai3-portal-billing — coordination log

(Newest entries at top. This file is written ONLY by AI3. I read every other
file in `coordination/` before starting any task.)

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
