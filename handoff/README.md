# handoff/

Cross-agent handoff notes. When one domain agent does work that another domain must
react to — a shared-file change, a new API contract, a scope addition, a data-shape
change — it drops a note here instead of editing the other agent's files.

## Why handoffs exist
The agents own disjoint slices of `servicem8-reports` (see the ownership table in
[../CLAUDE.md](../CLAUDE.md)). An agent must **never** edit another agent's files
directly — that's how parallel work clobbers itself. Instead it lands its own slice,
then hands off the follow-up.

## When to write one
- You changed a **shared file** (`servicem8.ts`, `accounts.ts`, `db.ts`, `storage.ts`)
  in a way another domain depends on.
- You added or changed an **API contract** another domain calls.
- You added an **SM8 scope** (every tenant must re-auth — everyone needs to know).
- You changed a **data shape** in the JSON store another domain reads/writes.
- You finished a piece that **unblocks** another agent's task.

## How to write one
Create `handoff/<YYYY-MM-DD>-<from>-to-<to>-<slug>.md` with this shape:

```markdown
---
from: platform
to: phone-bot
status: open            # open | acknowledged | done
pr: <staging PR # that landed the change, if any>
---

## What changed
One paragraph: the concrete change and where it landed (file + PR).

## What you need to do
The specific follow-up the receiving agent must make, with file paths.

## How to verify
What "correct" looks like on staging once both sides are in.
```

## Lifecycle
1. Author lands their slice on `staging`, then writes the handoff (`status: open`).
2. Receiving agent flips it to `acknowledged`, does the work on its own branch.
3. Once both sides are on `staging` and verified, flip to `done`.
4. Resolved handoffs stay in git history — don't delete them; they're the audit trail.

## Example
A real one from this ecosystem: `platform` enables sending SMS from jobs (adds
`publish_sms` to `SM8_SCOPES` and a `sm8SendSms` helper). That's a shared-file +
scope change, so it lands first with a handoff to `phone-bot` ("Riley can now send
the job-confirmation SMS via `sm8SendSms`; gate it on the tenant's *OK-to-text*
consent and the existing don't-notify checkbox"). `phone-bot` picks it up from the
note rather than `platform` reaching into `phone-bot.ts`.
