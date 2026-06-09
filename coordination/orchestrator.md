# orchestrator

**Role:** coordinate the three standing sessions (AI1/AI2/AI3), hold the whole-system
picture, and spin up subagents for cross-domain or overflow work. Does **not** edit a
domain's files while its AI is active — routes via a `→` note instead.

## Log (newest first)

### [2026-06-08] DONE — American-spelling sweep (product PR #264 → staging)
Prose-only British→American sweep: **168 word changes across 68 files**. Comment-scoped script
(only //, *, /* lines) + ~20 explicit UI-string fixes. Identifiers + external contracts (SM8
"Cancelled", Stripe "canceled", OAuth scopes, "not-authorised" error code, SM8-quoted
'not an authorised object type', {colour} test fixture) deliberately untouched. tsc clean · 763
tests · build green. Staging-only, NOT self-promoting (cosmetic; Dan can fold into the next
promote or take alone). PR #264 (draft). AI1's shared `accounts.ts` touched only for the 2-word
default-agreement prose "authorise"→"authorize".

### [2026-06-08] STARTING — American-spelling sweep (orchestrator direct; cross-cutting, prose-only)
Dan asked to convert English/Australian spellings → American across the platform (he spotted an
"authorise" with an s). Cross-domain cosmetic sweep; all three AIs idle, no single owner → I'm
doing it directly. Branch `chore/american-spelling` (off staging) in the product repo.
**Scope = PROSE ONLY** (Dan's pick): comments, JSDoc, and user-facing UI/strings. Method:
a comment-scoped script (only edits lines whose first non-ws is `//`/`*`/`/*`) + ~18 explicit
UI-string edits. **Identifiers left untouched** per Dan (cancelled flags, VOICE_CATALOGUE,
analyseRecurringJobs, applyCancellation, normalised local vars, JS Promise "fulfilled").
**Contracts EXCLUDED:** SM8 "Cancelled" status, Stripe "canceled", OAuth scopes, the
`"not-authorised"` error-code string, and the SM8-quoted `'not an authorised object type'` text.
Touches files across all four domains incl. shared `accounts.ts` (2-word default-agreement prose
"authorise"→"authorize") — claiming it here; AI1 idle. → PR vs staging, tsc+test+build green.

### [2026-06-08] DONE — PROMOTED staging→main (#255–#261) — ON PROD (Dan's go)
Dan verified staging and said "push everything to production." Promoted the whole stack via
**merge commit** (PR #262, merge `7382711`). Post-merge `git diff main staging` = **EMPTY**
(tree parity confirmed — no silently-dropped work). Railway auto-deploys main.

**Now LIVE on prod:**
- #255 header signed-in user + v1.12.0 + sm8-connection admin gate
- #256 transfer ownership · #257 customizable customer confirmation email
- **#258 + #259 OAuth login-bypass CLOSED** (hardened callback + removed /login Connect button +
  tightened legacy OAuth rule) — promoted TOGETHER as required; Dan verified password login /
  owner Reconnect / fresh-OAuth→/login / 2nd-tenant bootstrap on staging first.
- #260 client-side $-widget hiding · #261 "Admin" as a per-user permission / multiple admins

staging == main; nothing left to promote. The security bypass is closed in prod. All QUEUED
items from the prior brief are now shipped. Holding for Dan's next direction.

### [2026-06-08] STATE SYNC — onboarded; product access restored; whole OAuth-bypass batch on staging, holding for Dan's verify
Re-onboarded from all four coordination files. **Product repo access CONFIRMED** (`git ls-remote`
on `StillframeLLC/servicem8-reports` succeeds — the prior orchestrator lockout is gone).

Verified coordination logs against live PR state. **7 PRs sit on `staging`, NONE promoted to
`main`/prod** (`git log main..staging` = #255→#261, 24 files / +797/-89):
- #255 header signed-in user + v1.12.0 + sm8-connection admin gate (AI1)
- #256 transfer ownership (AI3)
- #257 customizable customer confirmation email (AI1)
- **#258 close OAuth login bypass — hardened callback (AI1)**  ⟵ SECURITY linchpin
- **#259 OAuth-bypass AI3 half — remove /login Connect button + tighten legacy rule**  ⟵ pairs w/ #258
- #260 client-side $-widget hiding (AI1) — was "QUEUED", now DONE on staging
- #261 "Admin" as a per-user permission / multiple admins (AI3) — was "QUEUED", now DONE on staging

**Net:** both items I had listed as QUEUED (#260, #261) already landed on staging. There is NO
open dispatch and NO idle-domain product work for me to pick up directly. The entire system is
blocked on **Dan's staging verify + promote signal**.

**Promotion gate (unchanged, critical):** #258 + #259 MUST promote together so Dan isn't locked
out. Before any promote, Dan verifies on staging: (1) password login works · (2) owner Reconnect
keeps him signed in · (3) fresh OAuth on his owner-tenant → `/login` (no full session) · (4) 2nd
tenant w/ no owner still bootstraps via OAuth. On his go: promote the whole #255→#261 stack
staging→main via MERGE commit + `git diff main staging` empty check. NEVER self-promote.

Holding for Dan's direction. No collisions; all three AIs idle per their last DONE entries.

### [2026-06-03] → ai3 + ai1 — "Admin" tab consolidation (Dan, placement locked)

Dan locked the UI. The owner-only **"Users" tab becomes the "Admin" tab** and absorbs the
SM8 reconnect/disconnect controls AND the Company Info content. Scope:

**→ ai3 (owns the tab, the `/advanced/users` page, and the AdvancedNav label):**
1. **Rename the tab "Users" → "Admin"** in `AdvancedNav.tsx` (still owner-gated via your
   `/api/account/me`, same gate as today). Route rename (`/advanced/users` →
   `/advanced/admin`) is your call — if you move it, redirect the old path.
2. **Host the SM8 Reconnect / Disconnect controls inside the Admin page**, wired to AI1's
   endpoint (spec in the entry below — disconnect clears token only, no "suspended"; no
   scope change).
3. **Move ALL "Company Info" tab contents into the Admin page**, then **remove the
   standalone "Company Info" tab from AdvancedNav.** Dan confirmed: Company Info is now
   **admin-only** — non-owner users lose access to it (that's intended, not a regression).

**Ownership flag — Company Info components:** if the Company Info panel/components are
platform-owned (AI1's `accounts.ts` `getDisplayCompanyNameOrFallback` etc. back it),
**AI3 must `→ platform` before editing AI1's files** — move the *rendering* into the Admin
page but don't unilaterally edit platform-owned data/components. AI1: confirm what's yours
under Company Info and either hand off the component or expose what AI3 needs. **AI1 + AI3:
agree the boundary in the log before AI3 edits anything shared.** AdvancedNav.tsx itself is
the shared nav AI3 already edits for tabs — fine to continue, just claim it.

**→ ai1 (platform):** unchanged from below — build the reconnect/disconnect endpoint +
token/suspend semantics; now you also know the UI lives in AI3's Admin page, so expose an
endpoint AI3 calls rather than building your own `/advanced` control.

Sequencing: AI1 endpoint + Company-Info boundary handoff first → AI3 wires UI + does the
rename/move. Promotions still gated on Dan's staging verify + go signal.

---

### [2026-06-03] → ai1 — GO: build SM8 Reconnect/Disconnect NOW (Dan confirmed)

Dan greenlit. **AI1: start the ServiceM8 Reconnect/Disconnect feature now** — don't wait
on the staging stack.

Confirmed spec from Dan:
- **Disconnect = clear the stored SM8 token only.** Do NOT trip the "suspended" state
  (it's user-initiated). No auto-suspend / no `markAccountDisconnected` side effects.
  Account stays active; tenant simply has no live token until they reconnect.
- **Reconnect = re-run OAuth on demand** (surface `/api/auth/login`) to get a fresh token.
- Reuse existing OAuth scopes — **no `SM8_SCOPES` change** (a scope change would force
  every tenant to re-auth; not wanted here).
- UI split: AI1 owns the endpoint(s) + token/suspend semantics; **AI3 offered to host the
  button on a page it owns** — AI1, settle that detail directly with AI3 (a "→ ai3" note),
  either expose an endpoint AI3 calls or own the `/advanced` control yourself.

**Promotions:** Dan drives staging verify himself; **he will tell the orchestrator when to
trigger AI1 to promote** the staging stack (#242/#243/#244) to prod. AI1: continue to NOT
self-promote until that signal comes through this log.

---

### [2026-06-03] DISPATCH + sequencing — after AI1/AI3 activity burst

Caught up on the new entries (AI1 #239/#240 verified→prod + #238 swept to prod; AI3 PR
#243 logout/create-login UX, PR #244 Phase-2 user mgmt — both staging; Phase-1 verified
by Dan on staging).

**→ ai1 (platform) — DISPATCH: take AI3's "→ platform" SM8 Reconnect/Disconnect request.**
Dan wants a tenant-facing "Reconnect ServiceM8" (re-run OAuth to refresh token) and a
"Disconnect" control in `/advanced`. It's platform-owned: SM8 OAuth (`/api/auth/login`),
token store (`accounts.ts`/`db.ts`), and the auto-suspend / `markAccountDisconnected`
logic must agree — a user-initiated disconnect must NOT trip the suspended state.
Suggested split (AI3 offered): **AI1 exposes the reconnect/disconnect endpoint(s) + owns
the token/suspend semantics; AI3 can host the button on a page it owns and call them** —
settle the UI-home detail directly with AI3. No new SM8 scopes expected (reuse existing
OAuth). This is the one open cross-domain action right now.

**→ ai3 — confirm the #238 partial-prod is OK.** AI1's blanket #241 staging→main sweep
carried #238 (tenant_users store + session fields) to PROD; #242 (login UI) stayed
staging-only. AI1 deems it harmless (additive, backend-only, unexercised without #242).
Your file still lists #238 as staging-only — please reconcile/ack. No action needed if
you're fine with #242 promoting on top later.

**Holding for Dan (verify + promote — AI1 is deliberately NOT self-promoting):**
staging has #242 (Phase-1 login), #243 (logout→/login + create-login UX), #244 (Phase-2
user management) stacked, all awaiting Dan's staging verify before staging→main.

No file collisions. PR #3 (this log) merges clean over the new main.

---

### [2026-06-03] CONSOLIDATED — all three STATE dumps received

All of AI1/AI2/AI3 are online, protocol-adopted, and idle (each "standing by").
PR #2 merged to `main` (resolved an add/add collision: the AIs had pushed their
STATE dumps straight to `main`, my branch seeded the same files — kept the real
dumps, took my README/orchestrator/§8). System picture:

**Where each domain is right now**
- **AI1 (platform):** clean. Recent prod ships: SM8 pagination fix (#224), serialized
  token refresh `getValidAccountToken` (#227/#228), dashboard freshness (#231),
  emergency SMS + preferred-appt toggle (#239), urgency toggle + passcode re-prompt
  (#240), all promoted (#241). No active WIP.
- **AI2 (phone bot):** clean code; one item *awaiting Dan*: `924feae` on prod needs a
  REAL test call to confirm transcript PDF attaches + stored-contact pull (last test
  used synthetic "test 73" with no contact records).
- **AI3 (portal/billing/auth):** Stripe live + trial-sync on prod. **Multi-user auth
  Phase 1 (#238 + #242) is on STAGING awaiting Dan's verify — NOT promoted to prod.**
  Phase 2 (user-mgmt UI) and Phase 3 (enforcement + server-side $-mask) not started.

**Cross-domain threads I'm tracking**
- **$-mask / permission enforcement (AI3 Phase 3)** WILL need a `→ platform` request
  (shared data paths / `access.ts` / dashboard). Sequence: platform lands shared hooks
  → AI3 builds enforcement. Not yet started; gated on Phase 1 verify.
- **Two-way inbound SMS to Riley (AI2)** depends on AI1's SM8 webhooks foundation.
  Deferred; coordinate platform-first when AI2 picks it up.
- **`servicem8.ts` `SM8_SCOPES` = `publish_sms`** already reconciled across AI1/AI2.
- **Backlog owned-but-stale:** checkout `successUrl`/`cancelUrl` override (AI1 asked AI3
  twice — small, AI3 lane); app-wide urgency default change (needs Dan yes/no).

**Blocking on Dan (verification, not code):**
1. Re-test call for Riley (`924feae`) → AI2 confirms transcript/contact.
2. Staging verify of multi-user auth Phase 1 login + owner bootstrap → unblocks AI3
   Phase 2/3 and the staging→prod promotion.

No collisions, nothing to dispatch yet. Holding for Dan's direction on what to drive next.

---

### [2026-06-03] STATE — coordination hub online
The `coordination/` folder is live as the team channel. Waiting on `STATE` dumps from
AI1, AI2, AI3 (append to your own file). Once I have all three, I'll post a consolidated
picture + any cross-domain sequencing here.

Standing rules I'll follow:
- I read all four files before dispatching anything.
- I never edit a domain's product files while its AI is running — I `→` the owning AI,
  or spin a subagent only after claiming the files here so we don't collide.
- Cross-domain features get split per domain and sequenced (shared-file changes land via
  platform first).
