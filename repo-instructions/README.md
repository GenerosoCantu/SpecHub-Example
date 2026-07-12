# Canonical Repo Instruction Files

This folder holds the **canonical copy** of each service repo's agent instruction file (`CLAUDE.md` for Claude Code; Copilot users point `.github/copilot-instructions.md` at it; Codex users use `AGENTS.md`). The per-repo instruction file is the one artifact that can silently drift from the hub's `CONVENTIONS.md` — keeping the canon here closes that gap.

## Rules

- One file per service repo: `{service}.md` (e.g. `main-api.md`, `admin-frontend.md`), using the service identifiers from `WORKFLOW.md`.
- Content is **stable, repo-wide conventions only** — stack, naming rules, module layout, env var names, plus the routing rule that points a fresh session at the right spec. Never feature-level detail; features reach a repo through a prompt, not through the instructions file.
- **Edit here first, then copy into the repo.** Never edit the file inside a service repo directly.
- `WORKFLOW.md` Step 5f: when a feature changes a shared convention, update `CONVENTIONS.md`, update the affected file(s) here, and copy them into the repo(s) in the same pass.

## In this example

Only `main-api.md` is filled in — it is the routing file the article excerpts. In a real hub there would be one file per service repo, seeded by copying each repo's current instruction file here, verifying it against `CONVENTIONS.md`, and resolving any drift in favor of the hub.
