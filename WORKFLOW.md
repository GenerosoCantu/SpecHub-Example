# Feature & Enhancement Workflow

**Last Updated:** 2026-07-11

End-to-end process for designing, implementing, and documenting any new feature or enhancement to the Plaza platform. Follow it in order — each step is a prerequisite for the next.

---

## Quick Reference

```
1. DESIGN   →  Create Features/FEATURE-{name}.md
2. CASCADE  →  Update relevant service spec(s) (01–05) + STATUS.md
3. PROMPT   →  Generate Prompts/PROMPT-{service}-{feature}.md (one per service)
4. IMPLEMENT → Apply prompts inside each service repo (new session per service)
5. CLOSE    →  Verify specs against the code, update STATUS.md + CHANGELOG, sync repo instructions, archive feature file & prompts
```

---

## Step 1 — Design the Feature

**Create `Features/FEATURE-{name}.md`**

Before touching any spec or writing any code, document the feature in the `Features/` folder. This file is the scratchpad where all design decisions are made before they become commitments.

A feature file should cover:

- **Description** — what the feature does from the user's perspective
- **Scope** — which services are affected (main-api, admin-frontend, etc.)
- **Data model** — field names, types, and indexes for any new schema
- **API shape** — endpoint paths, HTTP methods, guards, request/response shape
- **Frontend contract (when applicable)** — views/routes affected, component/state ownership, loading/empty/error states, and responsive behavior expectations
- **Backend contract (when applicable)** — module boundaries, schema/index changes, validation/auth rules, side effects (snapshot writes, events, jobs), and failure semantics
- **Open design decisions** — questions that need an answer before implementation can start. **Prompt generation must not run while any of these remain open.**
- **Implementation order (cross-service features)** — one line declaring the dependency order across services, e.g. `Implementation order: main-api → admin-frontend + storefront-frontend`. This line is carried into every generated prompt so sequencing is never implicit.
- **Recommended model (per service)** — for each affected service, the model tier to implement it with, plus a one-line reason. Default to the cheapest tier that fits; reserve the top tier for genuinely complex work (usually intricate UI). Example tiers with Claude: **Haiku** for simple/backend-only or config/data changes; **Sonnet** for standard features with moderate logic or light UI; **Opus** for complex UI stories. This recommendation is carried into each generated prompt's dispatch header (see Step 3).
- **Outstanding work** — a checklist of things still to build

> **Why a separate file?** The service spec files (`01–05`) describe the *implemented* state of the system. Mixing in-progress design with settled implementation facts pollutes the specs and makes prompt generation unreliable. The `Features/` file is the safe workspace for thinking; once decisions are final, they graduate into the spec. The feature file is archived to `Features/Implemented/` after implementation.

---

## Step 2 — Cascade into Service Specs

**Update `01-storefront-frontend.md`, `03-main-api.md`, etc.**

Once design decisions are confirmed, write the feature into the relevant service spec(s) as if it were already implemented. This means adding the module, endpoints, schema, Redux slice, views, and any storage changes in the precise format the rest of that spec uses.

Two bookkeeping moves complete the cascade:

- Add a new row for the feature to the `STATUS.md` board (status: Pending).
- Mark each not-yet-built spec section with a `<!-- PENDING: FEATURE-{name} -->` comment, so any reader — human or agent — knows it is designed but not yet real. Step 5 removes the markers once the work is verified.

> **Why write the spec before implementation?** The implementation prompt is generated *from* the spec. If the spec is vague or incomplete, the prompt will be too, and the agent will invent conventions. Writing the spec first forces every decision to be explicit — field names, validation rules, access control — which is exactly what the agent needs to produce correct code on the first pass.

---

## Step 3 — Generate Implementation Prompts

**Create `Prompts/PROMPT-{service}-{feature}.md` (one per affected service)**

Open a **new agent session** in this workspace and use the `generate-prompts` skill (`.claude/skills/generate-prompts/`), which encodes this step's required reads, preconditions, and structure. The agent reads `CONVENTIONS.md`, the feature file, and the affected service spec(s) first — never the whole hub.

Each prompt must be **stateless and self-contained**, and must open with this **dispatch header**:

```markdown
> **Target repo:** {path or repo name}
> **Branch:** {branch to create or work on}
> **Prerequisites:** {PROMPT-file(s) that must be applied and verified first, or "none"}
> **Status:** Generated   <!-- Generated → Applied → Verified -->
> **Recommended model:** {tier} — {one-line reason}
```

The status line makes `Prompts/` a dispatch board: `Generated` (not yet applied), `Applied` (implemented, awaiting human verification), `Verified` (verified in the repo; ready for Step 5). Update it as the prompt moves through Step 4.

| Prompt section | What it contains |
|---------------|-----------------|
| Files to study | Existing files in the service repo the agent should read for patterns (3–5 max) |
| Files to create | Exact file paths to create or modify |
| Schema contract | Field names, types, defaults, and indexes — no ambiguity |
| Endpoint definitions | Method, path, guard, request body, response shape |
| Pattern references | "Follow the same structure as `products.service.ts`" |
| Naming rules | DTO names, Redux slice name, action types, route strings — verbatim from the spec |

Additionally, include service-specific detail:

- **Backend prompts must specify** schema/index rules, DTO validation constraints, auth/guard behavior, endpoint error/status semantics, and required test updates.
- **Frontend prompts must specify** backend prerequisites/dependencies, route/view/component changes, state-management updates (Redux/query layer), UX states (loading/empty/error/permission), and acceptance criteria.

For cross-service features (e.g., a feature that requires both a Main API module and an Admin Frontend view), generate one prompt per service. The frontend prompt must explicitly declare the backend prompt as a prerequisite in its dispatch header's `Prerequisites:` line, and the feature file's `Implementation order` line must be reflected across the set of prompts.

> **Why one prompt per service?** Each service repo is a separate Git context. Applying everything in one prompt would require the agent to context-switch between two entirely different codebases in the same session, which degrades quality. Isolated prompts keep the agent focused on one codebase at a time and make the output easier to review.

> **Quality gate:** a prompt is complete only if a brand-new session in the target repo could implement it without asking a question or reading an unnamed file. The implementing session runs in a different workspace and cannot follow a reference back to a spec file — never cite hub paths in a prompt body.

---

## Step 4 — Implement

**Apply the prompts inside each service repo (new session per service)**

Open a **new agent session inside the target service repo** (the repo named in the prompt's dispatch header), on the model tier the prompt recommends, and paste or reference the generated prompt. Apply prompts in the feature's declared implementation order; a prompt whose prerequisites are not yet `Verified` must not be applied.

Rules for the implementation session:

- Start a fresh session for each service — do not carry a session across services.
- If the agent produces a bug or a missed edge case, fix it in **the same session** — the model has the full implementation in context and can correct precisely.
- If you need to revisit a different feature entirely, start a new session for that too.
- After applying a prompt, flip its dispatch-header `Status:` to `Applied`; after human verification (build runs, endpoints/UI exercised, diff read), flip it to `Verified`. Only `Verified` prompts graduate to Step 5.

> **Why a new session per service?** Context from a previous conversation — even a successfully completed one — introduces noise. The model may pattern-match on the wrong prior example or make assumptions about file state that are no longer true. A fresh session with a self-contained prompt produces cleaner, more predictable output.

---

## Step 5 — Close the Loop

After the implementation is verified, do all of the following **in a single pass** (open a new session in this Specs workspace and use the `close-loop` skill, `.claude/skills/close-loop/`, which encodes this checklist).

### 5a. Verify the spec against the code, then update the service spec(s)
- **Verify against the code, not memory.** For each affected service, open the implementation in the service repo itself (the path in the prompt's dispatch-header `Target repo:` line) and read the files the prompt named — schema, controller/routes, components. Confirm the spec's claimed contracts (field names, endpoint shapes, guards, component names) match what was actually built. Reality wins: reconcile the spec to the code, and record each deviation for the changelog entry.
- While in each service repo, capture the implementing commit SHA(s) or PR link(s) (`git log`) — they go into the changelog entry (5c).
- Remove the feature's `<!-- PENDING: FEATURE-{name} -->` markers.
- Update the `Last updated` header at the top of the spec file — keep it to a **single line** describing the current state. Do **not** append a per-feature log/history entry to the spec; that record belongs only in `CHANGELOG.md` (see the single-log rule below).

### 5b. Flip the status board (and overview only if needed)
- In `STATUS.md`, mark the feature row as complete (✅ for the relevant columns) and move it from **In Flight** to **Shipped**.
- **Keep the Notes cell to a single line.** The board is a status index, not a record — implementation detail belongs in `CHANGELOG.md` and the archived design doc, never in the Notes column.
- Touch `00-architecture-overview.md` only if the feature changed system-wide architecture (components, communication, tenancy, auth). If touched, bump its `Last Updated:` line.

### 5c. Update the changelog
- Prepend a dated entry to `CHANGELOG.md` describing what was implemented and what deviated from the design.
- Entries are listed newest-first, one per feature pass.
- **Include a traceability reference:** the implementing commit SHA(s) or PR link(s) in each affected service repo, e.g. `(main-api@a1b2c3d, admin-frontend#124)` — captured in 5a. This is what lets you reconstruct, months later, exactly which code a spec change produced.

> **Single-log rule.** `CHANGELOG.md` is the *only* document that accumulates dated log entries — for every stage worth recording (design, cascade, and close). All other documents stay clean and describe only their current state:
> - `00-architecture-overview.md` — header carries `Version` + a single `Last Updated:` line, nothing more.
> - Service specs (`01–05`) — each carries only its own `Last updated` line at the top.
> - Never paste a running history block into the overview, `STATUS.md`, or a service spec. If you catch one accumulating, fold the entries back into `CHANGELOG.md` and delete the block.

### 5d. Archive the feature design file
- Move `Features/FEATURE-{name}.md` to `Features/Implemented/`. Its *contracts* (schema, endpoints) now live in the service spec, but its *design rationale* (the "why", confirmed/open decisions, alternatives considered) is not captured anywhere else and is worth keeping.
- Add this banner to the top of the archived file so it is never mistaken for a live document:
  ```markdown
  > **ARCHIVED — historical design record. NOT a source of truth.**
  > Implemented YYYY-MM-DD. Current state lives in {service-spec}.md.
  > Do not edit; this captures the original design reasoning only.
  ```

### 5e. Archive the implementation prompt(s)
- Confirm each prompt's dispatch-header `Status:` is `Verified`, then move each `Prompts/PROMPT-{service}-{feature}.md` to `Prompts/Implemented/`.

### 5f. Sync repo instruction files if conventions changed
- If the feature changed any shared convention (naming rules, module layout, storage patterns, env conventions), update `CONVENTIONS.md` and the affected canonical repo instruction file(s) in `repo-instructions/`, then copy the updated file into the service repo. The hub copies are canonical; the per-repo instruction files must never drift from them.

> **Why archive instead of delete?** An archived feature file is a *frozen historical record*, not a living document — the banner makes that explicit, which neutralizes the risk of two diverging sources of truth. The service spec remains the only place describing *what currently exists*; the archived file preserves *how the decision was reached*.

> **Why do all of these in one pass?** Steps 5a–5f are all consequences of the same event — the feature being implemented. Doing them in one session ensures they are always consistent with each other. If you update the spec but forget to archive the feature file, the next person to open `Features/` will not know whether that feature is pending or already done.

---

## Naming Conventions

| Artifact | Convention | Example |
|----------|-----------|---------|
| Feature design file | `Features/FEATURE-{kebab-name}.md` | `Features/FEATURE-gift-cards.md` |
| Archived feature design file | `Features/Implemented/FEATURE-{kebab-name}.md` | `Features/Implemented/FEATURE-gift-cards.md` |
| Staled feature design file | `Features/Staled/FEATURE-{kebab-name}.md` | `Features/Staled/FEATURE-wishlists.md` |
| Implementation prompt | `Prompts/PROMPT-{service}-{feature}.md` | `Prompts/PROMPT-main-api-gift-cards.md` |
| Archived implementation prompt | `Prompts/Implemented/PROMPT-{service}-{feature}.md` | `Prompts/Implemented/PROMPT-main-api-gift-cards.md` |
| Service identifier in prompt name | `storefront-frontend`, `admin-frontend`, `main-api`, `tenant-service`, `asset-service` | — |

---

## Session Strategy Summary

| Situation | Session |
|-----------|---------|
| Designing a feature (writing the feature file) | Any session |
| Generating prompts from specs | New session in Specs workspace |
| Implementing a backend feature | New session in service repo |
| Implementing a frontend feature | New session in service repo (after backend) |
| Fixing a bug in the current implementation | Same session |
| Closing the loop (step 5) | New session in Specs workspace |
| Answering a question about what was just built | Same session |

---

## Staled Features

`Features/Staled/` holds feature designs that are **parked**: not being worked on, not implemented, but not discarded. It is a third lifecycle state alongside pending (`Features/`) and shipped (`Features/Implemented/`).

- **When to stale:** a pending feature is deprioritized indefinitely, superseded by another design, or blocked on a decision that will not be made soon. Move the file to `Features/Staled/` and add a one-line note at top: date staled + reason.
- **Status board:** update the feature's `STATUS.md` row to `⏸ Staled` (or remove the row if it never left design). Remove any `PENDING` markers the feature added to service specs during Cascade — staled designs must not linger in the source of truth.
- **Reviving:** a staled feature must be **re-validated before reuse**. The specs have moved since it was written; re-read the current spec section(s) it touches, correct the design against them, then move it back to `Features/` and restore its `STATUS.md` row to Pending. Never generate prompts directly from a staled file.

---

## Enhancements vs. Full Features

The same workflow applies to both. The difference is only in scope:

- **Enhancement** — modifies an existing module (new field, new endpoint, changed validation). Steps 1 and 2 may be very lightweight (a few lines in the feature file, a targeted spec edit). Generate a focused prompt that references the file being changed.
- **New feature** — new module, new schema, new views. All five steps apply in full.

Do not skip the feature file step for enhancements. Even a small change should be written down before generating a prompt — it forces the decision to be explicit and leaves a record of why the change was made.
