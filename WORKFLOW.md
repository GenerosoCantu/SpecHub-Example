# Feature & Enhancement Workflow

**Last Updated:** 2026-06-24

End-to-end process for designing, implementing, and documenting any feature on
the Plaza platform. Follow it in order — each step is a prerequisite for the
next. The gift-cards feature in `Features/` and `Prompts/` is a worked example of
steps 1–3.

---

## Quick Reference

```text
1. DESIGN    → Create Features/FEATURE-{name}.md
2. CASCADE   → Update relevant service spec(s) (01–05) + overview status table
3. PROMPT    → Generate Prompts/PROMPT-{service}-{feature}.md (one per service)
4. IMPLEMENT → Apply prompts inside each service repo (new session per service)
5. CLOSE     → Update specs + overview + CHANGELOG, archive feature file & prompts
```

---

## Step 1 — Design the Feature

**Create `Features/FEATURE-{name}.md`.**

Before touching any spec or writing code, capture the feature in `Features/`.
This is the scratchpad where design decisions are made before they become
commitments. In practice you describe the intent to the agent; the agent reads
the relevant service specs and drafts the file against them; you refine it.

Cover: intent, affected services, data model, API shape, the key design
decisions (and why), open questions, and what's out of scope.

> **Why a separate file?** The service specs (`01–05`) describe the *implemented*
> state. Mixing in-progress design into them pollutes the source of truth and
> makes prompt generation unreliable. The feature file is the safe place to
> think; settled decisions graduate into the specs in step 2.

---

## Step 2 — Cascade into Service Specs

**Update the affected `0X-*.md` specs, and `00-architecture-overview.md`.**

Write the agreed design into each affected service spec *as if already
implemented*, in that spec's format (module, endpoints, schema, slice, views).
Add a row to the Pending Features table (Section 12) of the overview, status
Pending.

> **Why spec before code?** The prompt is generated *from* the spec. A vague spec
> yields a vague prompt and the agent invents conventions. Writing the spec first
> forces every decision explicit — field names, validation, access control —
> which is exactly what produces correct code on the first pass.

---

## Step 3 — Generate Implementation Prompts

**Create `Prompts/PROMPT-{service}-{feature}.md`, one per affected service.**

In a fresh agent session in this hub, generate a prompt per service from its spec
+ the overview. Each prompt is stateless and self-contained: files to study,
files to create, exact schema contract, endpoint definitions, pattern
references, and verbatim naming rules. A frontend prompt names its backend prompt
as a prerequisite.

> **Why one prompt per service?** Each service repo is a separate Git context.
> One prompt spanning repos forces the agent to context-switch between codebases
> in a single session, degrading quality. Isolated prompts keep it focused and
> the output reviewable.

---

## Step 4 — Implement

**Apply each prompt in its own service repo, in a new session.**

Backend first; apply a dependent frontend prompt only after the backend is built
and tested. Fix any bug in the *same* session (the model has full context).
Never carry a session across services.

> **Why a new session per service?** Prior-conversation context — even from a
> success — is noise: the model may pattern-match the wrong example or assume
> stale file state. A fresh session + self-contained prompt is cleaner and more
> predictable.

---

## Step 5 — Close the Loop

After verifying the implementation, do all of the following in one pass (new
session in this hub):

- **5a.** Reconcile each service spec with what shipped; bump its `Last updated`.
- **5b.** Mark the feature ✅ in the overview's Pending Features table.
- **5c.** Prepend a line to `CHANGELOG.md`.
- **5d.** Move `Features/FEATURE-{name}.md` → `Features/Implemented/` with an
  `ARCHIVED — historical design record` banner.
- **5e.** Move applied prompts → `Prompts/Implemented/`.

> **Why archive, not delete?** The archived feature file is a frozen record of
> *how the decision was reached*; the banner keeps it from being mistaken for a
> source of truth. The spec remains the only description of *what currently
> exists*.
> **Why one pass?** 5a–5e are all consequences of the same event; doing them
> together keeps them consistent.

---

## Naming Conventions

| Artifact                 | Convention                                |
| ------------------------ | ----------------------------------------- |
| Feature design file      | `Features/FEATURE-{kebab-name}.md`        |
| Archived feature file    | `Features/Implemented/FEATURE-{kebab-name}.md` |
| Implementation prompt    | `Prompts/PROMPT-{service}-{feature}.md`   |
| Archived prompt          | `Prompts/Implemented/PROMPT-{service}-{feature}.md` |
| Service identifiers      | `storefront-frontend`, `admin-frontend`, `main-api`, `tenant-service`, `asset-service` |

---

## Enhancements vs. Full Features

Same workflow, different scope. An *enhancement* (new field/endpoint on an
existing module) makes steps 1–2 lightweight; a *new feature* (new module/schema/
views) uses all five steps in full. Do not skip the feature file even for small
changes — it forces the decision to be explicit and records why.
