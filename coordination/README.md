# coordination/ — the team whiteboard

The orchestrator and the three standing domain sessions (AI1, AI2, AI3) **don't talk
live** — separate Claude sessions can't. They coordinate here, asynchronously, through
git. This repo is the one place every session (including the orchestrator) can read and
write, so it's the shared channel.

## The golden rule: one file per session
Each session writes **only its own file**. That means there are never merge conflicts,
and nobody overwrites anyone. You read everyone's file before you start; you only ever
append to your own.

| File | Owner | Domain |
|------|-------|--------|
| `orchestrator.md` | the orchestrator | coordination, cross-domain dispatch, subagents |
| `ai1-platform.md` | AI1 | `platform` (+ shared servicem8/accounts/db/storage) |
| `ai2-phone-bot.md` | AI2 | `phone-bot` |
| `ai3-portal-billing.md` | AI3 | `customer-portal` + `stripe-billing` |

## Protocol (every session, every time)
1. **Before starting anything:** `git pull`, then read every file in this folder.
2. **Append to YOUR file only** (newest entry at top). Never touch another session's file.
3. **Entry types:**
   - `STATE` — a full snapshot of where your domain is.
   - `STARTING` — claiming work: what + which files you'll touch.
   - `DONE` — finished, with the product-repo PR #.
   - `→ <who>` — a request or handoff to the orchestrator or another AI.
   - `FYI` — anything the others should know.
4. **Code lives in the product repo** (`servicem8-reports`), each session on its own
   `feat/*` branch → PR. This repo is coordination only.

## Anti-collision rule
**One owner per file at a time.** Stay in your lane — only edit files your domain owns
(see [../CLAUDE.md](../CLAUDE.md)). Shared files (`servicem8.ts`, `accounts.ts`, `db.ts`,
`storage.ts`) belong to **platform/AI1**; everyone else requests changes with a
`→ platform` note rather than editing them. Before editing, claim with a `STARTING`
entry; if someone has an open claim on files you need, leave a `→` note and do something
else meanwhile.
