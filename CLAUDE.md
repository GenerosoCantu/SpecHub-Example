# Plaza Spec Hub — Agent Instructions

## Purpose

This workspace is a specification and prompt hub, not an application codebase. Treat the markdown documents here as the source of truth for platform behavior, contracts, workflow, and implementation handoff. Service repositories must not become alternate sources of truth: after implementation, the authoritative state is reconciled back into this workspace (WORKFLOW.md Step 5).

## Core Rules

- Always read the relevant spec file(s) before answering questions or generating output.
- Do not invent names, routes, DTOs, schema fields, Redux slice names, or workflow steps — prefer the exact naming and contracts already written in the specs.
- If a task affects process, prompt generation, feature lifecycle, or spec reconciliation, read `WORKFLOW.md` first.
- If a task affects system-wide behavior, read `00-architecture-overview.md` first.
- For shared conventions (API naming, entities, snapshot storage, env vars, logging, code organization), read `CONVENTIONS.md` — do not load the full overview just for conventions.
- For feature status, read `STATUS.md` — do not load the full overview just for status.
- If a task affects a specific service, read that service spec first.
- If required information is missing from the specs, say so and identify which document must be updated first.

## Required Reads By Task

- **General question about the platform** → `00-architecture-overview.md` (system-wide) or `CONVENTIONS.md` (conventions only), plus the relevant service spec.
- **Designing a feature** → `WORKFLOW.md`, `CONVENTIONS.md`, `STATUS.md`, the relevant service spec(s); then create/update `Features/FEATURE-{name}.md`.
- **Generating implementation prompts** → use the `generate-prompts` skill (`.claude/skills/generate-prompts/`); it encodes the required reads and the dispatch-header format.
- **Closing out an implemented feature** → use the `close-loop` skill (`.claude/skills/close-loop/`); it encodes the Step 5 checklist.
- **Reviewing/editing an existing prompt** → `WORKFLOW.md`, the relevant service spec, the target prompt file.

## File Map

- `00-architecture-overview.md` — system map and shared architectural decisions
- `CONVENTIONS.md` — shared cross-service conventions
- `STATUS.md` — live feature status board (flipped in Step 5)
- `WORKFLOW.md` — authoritative process for Design → Cascade → Prompt → Implement → Close
- `CHANGELOG.md` — implementation history (the only document that accumulates dated entries)
- `SPEC-GUIDELINES.md` — what goes in the overview vs. a service spec
- `01…05-*.md` — one spec per service
- `Features/` — pending feature designs; `Features/Implemented/` — archived (frozen); `Features/Staled/` — parked (see WORKFLOW.md)
- `Prompts/` — active implementation prompts (each carries a dispatch header); `Prompts/Implemented/` — prompts for completed work
- `repo-instructions/` — canonical copies of each service repo's instruction file, synced in Step 5f
- `templates/` — fill-in templates (spec, feature, prompt)
- `.claude/skills/` — the workflow's recurring motions, packaged as skills

## Session Guidance

- Prompt generation happens in a fresh session in this hub; implementation in a fresh session inside the target service repo; close-out in a fresh session back here. Bug fixes for an in-progress implementation stay in the same service-repo session.

> **Note for Copilot users:** `.github/copilot-instructions.md` is a pointer to this file. This file is the single source of truth for workspace instructions.
