# Feature & Enhancement Workflow

**Last Updated:** 2026-06-27

End-to-end process for designing, implementing, and documenting any new feature or enhancement to the Plaza platform. Follow it in order — each step is a prerequisite for the next.

---

## Quick Reference

```
1. DESIGN   →  Create Features/FEATURE-{name}.md
2. CASCADE  →  Update relevant service spec(s) (01–05)
3. PROMPT   →  Generate Prompts/PROMPT-{service}-{feature}.md (one per service)
4. IMPLEMENT → Apply prompts inside each service repo (new session per service)
5. CLOSE    →  Update specs + architecture overview + CHANGELOG, archive feature file & prompts
```

---

## Step 1 — Design the Feature

**Create `Features/FEATURE-{name}.md`**

Before touching any spec or writing any code, document the feature in the `Features/` folder. This file is the scratchpad where all design decisions are made before they become commitments.

A feature file should cover:

- **Description** — what the feature does from the user's perspective
- **Scope** — which services are affected (Main API, Storefront Frontend, etc.)
- **Data model** — field names, types, and indexes for any new schema
- **API shape** — endpoint paths, HTTP methods, guards, request/response shape
- **Frontend contract (when applicable)** — views/routes affected, component/state ownership, loading/empty/error states, and responsive behavior expectations
- **Backend contract (when applicable)** — module boundaries, schema/index changes, validation/auth rules, side effects (storage writes, events, jobs), and failure semantics
- **Open design decisions** — questions that need an answer before implementation can start
- **Outstanding work** — a checklist of things still to build

> **Why a separate file?** The service spec files (`01–05`) describe the *implemented* state of the system. Mixing in-progress design with settled implementation facts pollutes the specs and makes prompt generation unreliable. The `Features/` file is the safe workspace for thinking; once decisions are final, they graduate into the spec. The feature file is archived to `Features/Implemented/` after implementation.

---

## Step 2 — Cascade into Service Specs

**Update `01-storefront-frontend.md`, `02-main-api.md`, etc.**

Once design decisions are confirmed, write the feature into the relevant service spec(s) as if it were already implemented. This means adding the module, endpoints, schema, Redux slice, views, and any storage changes in the precise format the rest of that spec uses.

Also update `00-architecture-overview.md` Section 12 status table to add a new row for the feature (status: Pending).

> **Why write the spec before implementation?** The implementation prompt is generated *from* the spec. If the spec is vague or incomplete, the prompt will be too, and the agent will invent conventions. Writing the spec first forces every decision to be explicit — field names, validation rules, access control — which is exactly what the agent needs to produce correct code on the first pass.

---

## Step 3 — Generate Implementation Prompts

**Create `Prompts/PROMPT-{service}-{feature}.md` (one per affected service)**

Open a **new agent session** in this workspace. Ask the agent to generate an implementation prompt for each affected service, reading the relevant service spec and `00-architecture-overview.md` first.

Each prompt must be **stateless and self-contained**:

| Prompt section | What it contains |
|---------------|-----------------|
| Files to study | Existing files in the service repo the agent should read for patterns |
| Files to create | Exact file paths to create or modify |
| Schema contract | Field names, types, defaults, and indexes — no ambiguity |
| Endpoint definitions | Method, path, guard, request body, response shape |
| Pattern references | "Follow the same structure as `galleries.service.ts`" |
| Naming rules | DTO names, Redux slice name, action types, route strings — verbatim from the spec |

Additionally, include service-specific detail:

- **Backend prompts must specify** schema/index rules, DTO validation constraints, auth/guard behavior, endpoint error/status semantics, and required test updates.
- **Frontend prompts must specify** backend prerequisites/dependencies, route/view/component changes, state-management updates (Redux/query layer), UX states (loading/empty/error/permission), and acceptance criteria.

For cross-service features (e.g., a feature that requires both a Main API module and a Storefront Frontend view), generate one prompt per service. The frontend prompt must explicitly declare the backend prompt as a prerequisite.

> **Why one prompt per service?** Each service repo is a separate Git context. Applying everything in one prompt would require the agent to context-switch between two entirely different codebases in the same session, which degrades quality. Isolated prompts keep the agent focused on one codebase at a time and make the output easier to review.

---

## Step 4 — Implement

**Apply the prompts inside each service repo (new session per service)**

Open a **new agent session inside the target service repo** and paste or reference the generated prompt. Apply the backend prompt first; apply the frontend prompt only after the backend is implemented and tested.

Rules for the implementation session:

- Start a fresh session for each service — do not carry a session across services.
- If the agent produces a bug or a missed edge case, fix it in **the same session** — the model has the full implementation in context and can correct precisely.
- If you need to revisit a different feature entirely, start a new session for that too.

> **Why a new session per service?** Context from a previous conversation — even a successfully completed one — introduces noise. The model may pattern-match on the wrong prior example or make assumptions about file state that are no longer true. A fresh session with a self-contained prompt produces cleaner, more predictable output.

---

## Step 5 — Close the Loop

After the implementation is verified, do all of the following **in a single pass** (open a new session in this Specs workspace):

### 5a. Update the service spec(s)
- Confirm the spec matches what was actually built (field names, endpoint shapes, frontend component names). Correct any discrepancies.
- Update the `Last updated` header at the top of the spec file — keep it to a **single line** describing the current state. Do **not** append a per-feature log/history entry to the spec; that record belongs only in `CHANGELOG.md` (see the single-log rule below).

### 5b. Update the architecture overview
- In `00-architecture-overview.md` Section 12 status table, mark the feature row as complete (✅ for the relevant columns).
- Bump the `Last Updated:` line in the overview header.
- **Do not add a log/history entry to the overview.** The overview describes the *current* system, not its history. Its header carries only `Version` and a single `Last Updated:` line; the running log lives in `CHANGELOG.md` only (see the single-log rule below).

### 5c. Update the changelog
- Prepend a one-line entry to `CHANGELOG.md` describing what was implemented.
- Entries are listed newest-first, one per feature pass.

> **Single-log rule.** `CHANGELOG.md` is the *only* document that accumulates dated log entries. All other documents stay clean and describe only their current state:
> - `00-architecture-overview.md` — header carries `Version` + a single `Last Updated:` line, nothing more.
> - Service specs (`01–05`) — each carries only its own `Last updated` line at the top.
> - Never paste a running history block into the overview or a service spec. If you catch one accumulating, fold the entries back into `CHANGELOG.md` and delete the block.

### 5d. Archive the feature design file
- Move `Features/FEATURE-{name}.md` to `Features/Implemented/`. Its *contracts* (schema, endpoints) now live in the service spec, but its *design rationale* (the "why", confirmed/open decisions, alternatives considered) is not captured anywhere else and is worth keeping.
- Add this banner to the top of the archived file so it is never mistaken for a live document:
  ```markdown
  > **ARCHIVED — historical design record. NOT a source of truth.**
  > Implemented YYYY-MM-DD. Current state lives in {service-spec}.md.
  > Do not edit; this captures the original design reasoning only.
  ```

### 5e. Archive the implementation prompt(s)
- Move each applied `Prompts/PROMPT-{service}-{feature}.md` to `Prompts/Implemented/`.

> **Why archive instead of delete?** An archived feature file is a *frozen historical record*, not a living document — the banner makes that explicit. The service spec remains the only place describing *what currently exists*; the archived file preserves *how the decision was reached*.

> **Why do all of these in one pass?** Steps 5a–5e are all consequences of the same event — the feature being implemented. Doing them in one session ensures they are always consistent with each other.

---

## Naming Conventions

| Artifact | Convention | Example |
|----------|-----------|---------|
| Feature design file | `Features/FEATURE-{kebab-name}.md` | `Features/FEATURE-gift-cards.md` |
| Archived feature design file | `Features/Implemented/FEATURE-{kebab-name}.md` | `Features/Implemented/FEATURE-gift-cards.md` |
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

## Enhancements vs. Full Features

The same workflow applies to both. The difference is only in scope:

- **Enhancement** — modifies an existing module (new field, new endpoint, changed validation). Steps 1 and 2 may be very lightweight (a few lines in the feature file, a targeted spec edit). Generate a focused prompt that references the file being changed.
- **New feature** — new module, new schema, new views. All five steps apply in full.

Do not skip the feature file step for enhancements. Even a small change should be written down before generating a prompt — it forces the decision to be explicit and leaves a record of why the change was made.
