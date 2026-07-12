# Plaza Specification Guidelines

The canonical checklist for what goes in the architecture overview
(`00-architecture-overview.md`) and in each service spec (`01-…` … `05-…`). Use
this when writing or updating any Plaza spec. The fill-in scaffolds live in
`templates/`.

---

## 1. Architecture Overview (`00-architecture-overview.md`)

The single source of truth for the whole platform: a high-level description plus
a navigation/status hub. **Required sections:**

1. **Version & Last Updated** — version field + a **single** `Last Updated:`
   line, nothing more. Dated history belongs only in `CHANGELOG.md`
   (single-log rule, `WORKFLOW.md` Step 5).
2. **Table of Contents** — numbered, covering all major sections.
3. **Product Summary** — one paragraph: what the platform is and who it serves.
4. **System Components** — deployable units grouped as client apps / backend
   services / data, each with stack, port, hosting, purpose.
5. **High-Level Architecture Diagram** — ASCII/Mermaid showing request flow.
6. **Multi-Tenancy Strategy** — tenant resolution, isolation, static delivery.
7. **Authentication & Authorization** — token flow, claims, delegated validation.
8. **Service Communication** — who calls whom; public vs authenticated.
9. **Hosting & Deployment** — where each unit runs.
10. **Shared Conventions** — a pointer to `CONVENTIONS.md`, which holds the
    system-wide rules (API naming, entities, storage model, env-var prefixes,
    logging, code organization, IDs). Extracted so an agent checking a naming
    rule doesn't load the whole overview.
11. **Spec Document Index** — one line per service spec.
12. **Known Gaps & Technical Debt** — honest limitations.
13. **Open Questions Log** — open decisions, with a blocking flag.
14. **Feature Status** — a pointer to `STATUS.md`, the live status board
    (extracted for the same reason as conventions).

**Keep it high-level.** No full endpoint reference, DTO schemas, or Redux slice
names here — those belong in the service specs.

**Every distinct kind of question gets the smallest file that can answer it.**
`CONVENTIONS.md` and `STATUS.md` started life as overview sections and were
extracted the day an agent had to load the full architecture document just to
check a naming rule or a feature's status.

---

## 2. Service Specs (`01-…` … `05-…`)

Each reads like a stable design doc for its service — the *implemented* state,
not a draft. Two scaffolds back this section: copy
`templates/SERVICE-SPEC-TEMPLATE.md` for a **backend** service (endpoints, data
model, server-side auth) and `templates/FRONTEND-SPEC-TEMPLATE.md` for a
**frontend / client app** (state, views, API consumption). The common structure
below applies to both; frontends swap the endpoint-shaped sections for the
frontend-shaped ones noted under *Service-specific expectations*. **Common
structure:**

1. **Header** — title, one-line description, service identifier, `Last updated`.
2. **Table of Contents.**
3. **Overview** — purpose; what it owns and what it explicitly does not.
4. **Tech Stack.**
5. **Architecture** — module/component layout; a tree or diagram.
6. **Authentication** — strategy, guards, public vs protected, delegation.
7. **Data Model(s)** — field-by-field per entity: type, required, default, index.
8. **Endpoints** — grouped public/protected; per endpoint: method, path, guard,
   request/DTO, response, side effects.
9. **Endpoint Summary** — quick-reference table.
10. **Environment Variables.**
11. **Cross-Cutting Concerns / Patterns** — middleware, interceptors, filters; for
    frontends, the base request layer and Redux patterns.
12. **Key Design Notes** — decisions worth remembering.
13. **Known Issues & Gaps** — caveats and unresolved items (never inline guesses).

### Service-specific expectations

- **`01-storefront-frontend.md`** — snapshot consumption, what's static vs live,
  checkout against the public API.
- **`02-admin-frontend.md`** — base request layer, Redux slices, API services,
  views, the create/edit view pattern.
- **`03-main-api.md`** — full BFF design: modules, auth lifecycle, every endpoint,
  the publish/snapshot flow, ID and lifecycle rules.
- **`04-tenant-service.md`** — lightweight: store resolution, the `stores` model,
  auth delegation.
- **`05-asset-service.md`** — storage layout, upload/snapshot/delete contracts,
  auth delegation.

### Tone & rules

- Describe the implemented state; use exact identifiers; prefer tables and
  request/response examples over prose; put anything unresolved in *Known Issues*
  or the overview's *Open Questions*, never inline as if real.
- **Cascaded-but-not-built sections carry a `<!-- PENDING: FEATURE-{name} -->`
  marker** (added in step 2, removed in step 5 once the work is verified), so
  any reader — human or agent — knows the section is designed but not yet real.
- Each spec's header carries a **single** `Last updated` line describing current
  state; never a running history block (single-log rule — history goes in
  `CHANGELOG.md`).

### When a spec outgrows the window: split specs

Spec-as-compression buys a long runway, not an infinite one. When a service
spec grows past the point where "read the spec" means dragging thousands of
lines into context to answer a question about one module, apply the same
compression one level down — **the spec file becomes an index over per-module
files:**

```text
03-main-api.md            ← index: ~50 lines, a table of modules
03-main-api/
├── 00-core.md            ← stack, auth, cross-cutting behavior
├── gift-cards.md         ← one file per module
├── products.md
└── …
```

The routing rule does the real work: for any module task, the agent reads the
**index**, the directory's **core file**, and **only the module file(s) the
task touches** — never the whole directory. A new module gets its own file plus
one row in the index table; module content never migrates back into the index.
Context load per task then stays roughly constant as the system grows.

Plaza's five specs all still fit comfortably in single files, so none are split
here — split only when you feel the friction, not in advance.

---

## 3. Overview vs. Service Specs

The overview describes structure, relationships, and system-wide patterns and
links out to specs; it never duplicates service-level endpoint detail. When in
doubt, detailed contracts go in the service spec.

**When to update which:** designing a feature → add a Pending row in `STATUS.md`
and cascade into the affected specs with `PENDING` markers (step 2). Shipping it
→ verify the specs against the code and reconcile, remove the markers, flip the
`STATUS.md` row, add a dated `CHANGELOG.md` entry with commit refs, archive
(step 5).

---

## 4. Naming Conventions

- Feature files: `Features/FEATURE-{kebab-name}.md`
- Spec files: `01-storefront-frontend.md` … `05-asset-service.md`
- Prompt files: `Prompts/PROMPT-{service}-{feature}.md`
- Service identifiers: `storefront-frontend`, `admin-frontend`, `main-api`,
  `tenant-service`, `asset-service`

---

## 5. How to Use These Guidelines

1. Read `WORKFLOW.md` first for the feature lifecycle.
2. Copy a scaffold from `templates/` and fill it against the real source.
3. Keep the overview high-level and the service specs implementation-specific.
4. Keep all service specs consistent in structure and level of detail.
