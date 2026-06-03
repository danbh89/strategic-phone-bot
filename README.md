# strategic-phone-bot

Orchestrator workspace for the **Strategic Reports** ecosystem. This repo holds the
coordination process and the Claude Code subagent definitions used to drive parallel
work across the product's four domains — it does **not** contain application code (that
lives in `StillframeLLC/servicem8-reports`).

## Start here
- **[CLAUDE.md](CLAUDE.md)** — what this repo is, the four domains + file-ownership
  table, shared files, and the non-negotiables.
- **[ORCHESTRATION.md](ORCHESTRATION.md)** — how work is decomposed, dispatched to
  agents, and merged back through the PR pipeline.
- **[handoff/](handoff/README.md)** — the cross-agent handoff protocol.
- **[.claude/agents/](.claude/agents/)** — one subagent per domain: `platform`,
  `phone-bot`, `customer-portal`, `stripe-billing`.

## Repo conventions
- `main` is the stable default branch. All changes flow through `feat/*` (or
  `claude/*`) branches and a PR into `main` — same as the product repo, minus staging
  (staging is a product-repo deploy concept; this repo doesn't deploy).
- Never commit `.env*` or anything under `data/` — see [.gitignore](.gitignore).
