# ai2-phone-bot — coordination log

(Newest entries at top. This file is written ONLY by AI2. I read every other
file in `coordination/` before starting any task.)

### [2026-06-03] AI2 online — protocol adopted
Standing session. Cloned `../coord`, read `coordination/ai1-platform.md` (only peer file
present). Will pull + read all files before each task, append newest-at-top to this file
only, claim files with a STARTING entry before editing, stay in the phone-bot lane, and
route shared-file (`servicem8.ts`/`accounts.ts`/`db.ts`/`storage.ts`) changes through a
"→ platform" note. Continuing current phone-bot work under these rules. Standing by.

---

### [2026-06-03] STATE — AI2 (phone bot / "Riley")

**Domain + files I work in**
- The phone bot ("Riley") — a single shared Telnyx AI assistant morphed per-tenant.
  Product repo: `StillframeLLC/servicem8-reports`.
- Owned files:
  - `src/app/api/sm8/telnyx-webhook/route.ts` (Telnyx webhook: dynamic-vars, tool calls,
    job creation, end-of-call transcript/notify, two-way SMS routing)
  - `src/lib/phone-bot.ts`, `src/lib/phone-bot-submit.ts`, `src/lib/phone-bot-caller.ts`
  - `src/app/advanced/phone-bot/`, `src/app/advanced/phone-bot-setup/`
  - `src/components/advanced/PhoneBotEditor.tsx`, `PhoneBotSetupWizard.tsx`
  - `src/app/api/account/phone-bot-config/route.ts`
  - `src/app/api/admin/accounts/[id]/phone-bot-numbers/route.ts`
  - `scripts/phone-bot/` (committed `assistant-config.json` source-of-truth + `sync-assistant.mjs`)

**Shared code I DEPEND on (owned by AI1 — I request via "→ platform"):**
- `getValidAccountToken` (accounts.ts) — phone-bot + portal now route token refresh
  through this central serialized helper (fixed the rotating-refresh-token race).
- `resolveCustomerUuid` (embed-customer-match.ts) — silent tier-1/2/3 customer matching;
  phone-bot-submit mirrors the embed flow and calls it.
- `sm8Fetch`/`sm8Post`/`sm8SendSms`/`sm8UploadFile` + `SM8_SCOPES` (servicem8.ts),
  `getPublicJobConfig`/`getDisplayCompanyNameOrFallback` (accounts.ts).

**Done (shipped to prod/main today)**
- #230 promote token-race fix: phone-bot + portal delegate to `getValidAccountToken`
  (was the root cause of recurring mid-call disconnects / 401 invalid_token).
- #233 no-job follow-up email: stop false-positive (Telnyx `conversation_ended` carries a
  different `call_control_id` than AI-leg tool calls → switched to a recent created-job
  call-log window) + pull matched customer's on-file address onto the job.
- #235 attach matched customer's stored primary contact to the job (caller stays primary).
- #237 recover jobUuid at end-of-call from the call log so the transcript PDF + diary note
  land on the job (same ccid-mismatch root cause) + stored-contact diagnostics.
- Earlier this session: SMS staff prefix "Riley - {company}:", in-flow job-created
  notifications (email+SMS customer confirm; staff SMS only on no-job calls), ANI capture
  at dynamic-vars, fail-closed Telnyx signature check (SECURITY-TODO #10), lookupCustomerJobs
  + staff-gated lookupExistingJob, Advanced-Tools home page + Phone Bot tab + setup wizard,
  committed assistant config + idempotent sync script.

**In progress / awaiting verification**
- `924feae` live on prod — awaiting Dan's re-test call to confirm transcript PDF now lands
  and stored-contact pull works against a REAL customer (last test used synthetic "test 73"
  which has no company-contact records → nothing to pull; new diagnostics will confirm).

**Open bugs / known issues**
- Riley's speech-to-text mishears spoken contact details (email/name). Option: tune the
  assistant prompt to read details back for confirmation. Not yet done.
- Call AUDIO recording → diary is NOT implemented (only the transcript PDF ever was).
  Would need Telnyx call recording enabled + fetching the recording URL on call end.
  Offered to Dan; awaiting go-ahead.
- Two-way SMS (customer texting Riley back) deferred — depends on AI1's SM8 webhooks
  foundation; SM8 `sm8SendSms` currently sends with `regardingJobUuid` but inbound reply
  routing isn't wired.

**Integrations I rely on (env var NAMES only — no secrets)**
- Telnyx: `TELNYX_API_KEY` (assistant/SMS), `TELNYX_PUBLIC_KEY` (webhook signature verify —
  was missing on prod, now set). Single "Riley" assistant; tool URLs point at ONE host
  (currently prod via `sync:prod`). Per-call tenant binding via in-memory `callContextMap`
  keyed by `call_control_id`, captured at the dynamic-vars step.
- ServiceM8: outbound SMS via `sm8SendSms` (needs `publish_sms` scope — now in SM8_SCOPES);
  job create / attachments / contacts via the sm8 helpers; token via `getValidAccountToken`.
- Email: `RESEND_API_KEY`.

**→ platform (FYI, already merged — no action needed):** earlier this session `servicem8.ts`
got `SM8_SCOPES` bumped to include `publish_sms` + a `logGrantedScopes()` diagnostic on
token exchange/refresh. AI1's STATE already reflects `publish_sms`, so this is reconciled.
Going forward I'll route any servicem8.ts change through a "→ platform" note.

**Next 3-5 tasks (priority order)**
1. After Dan's next test call: read prod logs, confirm transcript PDF attaches + stored-contact
   diagnostic outcome; report.
2. If Dan green-lights: scope + build call AUDIO recording → diary attachment.
3. If recurring: tune Riley's prompt to confirm spoken contact details back to the caller.
4. Two-way inbound SMS to Riley — coordinate with AI1 once SM8 webhooks foundation lands
   (→ platform when ready).
5. Migrate remaining lower-frequency token racers (account/admin backup routes still using
   `session.refreshToken`) — coordinate with AI1 (→ platform if accounts.ts touched).
