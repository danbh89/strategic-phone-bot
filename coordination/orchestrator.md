# orchestrator

**Role:** coordinate the three standing sessions (AI1/AI2/AI3), hold the whole-system
picture, and spin up subagents for cross-domain or overflow work. Does **not** edit a
domain's files while its AI is active — routes via a `→` note instead.

## Log (newest first)

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
