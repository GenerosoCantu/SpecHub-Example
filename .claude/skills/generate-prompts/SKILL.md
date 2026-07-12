---
name: generate-prompts
description: Generate stateless implementation prompts from the Plaza specs (WORKFLOW.md Step 3). Use when the user asks to generate, create, or regenerate a PROMPT-{service}-{feature}.md file, or says "step 3" / "generate the prompts" for a designed feature.
---

# Generate Implementation Prompts (Step 3)

Produce one stateless, self-contained prompt per affected service for a feature that has completed Design (Step 1) and Cascade (Step 2).

## 1. Required reads — in this order, nothing more

1. `CONVENTIONS.md` — shared conventions every prompt inherits
2. `Features/FEATURE-{name}.md` — the feature file (must exist; if it doesn't, stop and run Step 1 first)
3. For each affected service: its service spec (`01-storefront-frontend.md` … `05-asset-service.md`)
4. `STATUS.md` — confirm the feature row exists and is Pending

Do NOT read the full `00-architecture-overview.md` or unrelated service specs.

## 2. Preconditions — verify before generating

- The feature file has no unresolved items under "Open design decisions". If any remain, stop and list them to the user.
- The Cascade is done: the contracts appear in the affected spec(s) with `<!-- PENDING: FEATURE-{name} -->` markers. If not, stop — the prompt is generated *from the spec*, not from the feature file.
- Cross-service features: the feature file declares an `Implementation order` line.

## 3. Write one prompt per affected service

File: `Prompts/PROMPT-{service}-{feature}.md`. Service identifiers: `storefront-frontend`, `admin-frontend`, `main-api`, `tenant-service`, `asset-service`.

Every prompt MUST open with the dispatch header:

```markdown
> **Target repo:** {repo path/name}
> **Branch:** {branch name, e.g. feature/{kebab-name}}
> **Prerequisites:** {PROMPT-file(s) that must be Verified first, or "none"}
> **Status:** Generated   <!-- Generated → Applied → Verified -->
> **Recommended model:** {tier} — {one-line reason, carried from the feature file}
```

Then the body, per WORKFLOW.md Step 3:

- **Context** — what the feature does, in 2–5 sentences
- **Files to study** — existing repo files to read for patterns (keep it to 3–5)
- **Files to create/modify** — exact paths
- **Schema contract** — field names, types, defaults, indexes, verbatim from the spec
- **Endpoint definitions** — method, path, guard, request/response shape
- **Pattern references** — "follow the same structure as X"
- **Naming rules** — DTO names, Redux slice, route strings, verbatim from the spec
- Backend prompts: DTO validation, auth/guard behavior, error/status semantics, required test updates
- Frontend prompts: backend prerequisites, route/view/component changes, state updates, UX states (loading/empty/error/permission), acceptance criteria

## 4. Quality gate

A prompt is complete only if a brand-new session in the target repo could implement it without asking a question or reading an unnamed file. Re-read each prompt against that bar before finishing. Do not include spec-hub file paths in the prompt body — the implementing session has no access to this workspace.

## 5. Finish

Report the generated prompt files and the order in which they must be applied.
