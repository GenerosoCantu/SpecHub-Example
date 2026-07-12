# Plaza Spec Hub — a worked example of the SpecHub multi-repo AI workflow

This repository is the **companion example** to the two-part SpecHub article
series: *"SpecHub, Part 1: How I Ship Features Across Three Repos With an AI
Agent (and Stay Sane)"* and *"SpecHub, Part 2: What Fifty Features Taught Me
About Running a Spec-First AI Workflow."* It is a small, **fictional**
multi-tenant storefront platform ("Plaza") used only to demonstrate the
workflow — none of it is a real product.

The point of the repo is not the product. It is the **structure**: how specs,
conventions, a status board, feature files, prompts, skills, and instruction
files fit together so an AI agent can ship a feature that spans several
services without drifting or losing context.

## The fictional system

Plaza is a multi-tenant storefront platform. Each tenant is a *store* with its
own public site. It is built from five deployable units plus a shared database:

| Service                 | Role                                              | Stack       |
| ----------------------- | ------------------------------------------------- | ----------- |
| `storefront-frontend`   | Public buyer-facing storefront (per store)        | Next.js     |
| `admin-frontend`        | Seller dashboard to manage a store                | React+Redux |
| `main-api`              | Backend-for-frontend; owns products, orders, etc. | NestJS      |
| `tenant-service`        | Resolves a store by domain; owns the store record | NestJS      |
| `asset-service`         | Image uploads + published JSON snapshots (CDN)     | NestJS      |
| MongoDB Atlas           | Shared datastore (`plaza` database)               | —           |

**This is the Spec Hub.** The five service repos are not in this repo — they
would be separate repositories containing only source code plus a lean
instruction file (`CLAUDE.md`, `.github/copilot-instructions.md`, or
`AGENTS.md`, depending on your agent). The canonical copy of each of those
instruction files lives *here*, in `repo-instructions/`, so the one artifact
that could silently drift stays under the hub's control.

## What's in here

```text
.
├── 00-architecture-overview.md     ← the map; first stop for any system-wide question
├── CONVENTIONS.md                  ← shared cross-service conventions (extracted from the overview)
├── STATUS.md                       ← live feature status board (one-line notes only)
├── WORKFLOW.md                     ← the feature lifecycle (the 5 steps) — authoritative
├── CHANGELOG.md                    ← implementation history (the ONLY dated log in the hub)
├── SPEC-GUIDELINES.md              ← what goes in a spec, PENDING markers, the split-spec pattern
├── CLAUDE.md                       ← the hub's own agent instructions (routing rules)
├── 01-storefront-frontend.md       ← service specs (one per service)
├── 02-admin-frontend.md
├── 03-main-api.md                  ← the fullest example spec
├── 04-tenant-service.md
├── 05-asset-service.md
├── Features/
│   ├── FEATURE-gift-cards.md       ← a worked, cross-service design scratchpad
│   ├── Implemented/                ← archived feature files for shipped work (frozen records)
│   └── Staled/                     ← parked designs — not active, not implemented
├── Prompts/                        ← a dispatch board: each prompt carries a status header
│   ├── PROMPT-main-api-gift-cards.md
│   ├── PROMPT-admin-frontend-gift-cards.md      ← declares the main-api prompt as prerequisite
│   ├── PROMPT-storefront-frontend-gift-cards.md ← declares the main-api prompt as prerequisite
│   └── Implemented/                ← prompts for completed work
├── repo-instructions/              ← canonical copies of each service repo's instruction file
├── templates/
│   ├── SERVICE-SPEC-TEMPLATE.md    ← fill-in skeleton for a backend service spec
│   ├── FRONTEND-SPEC-TEMPLATE.md   ← fill-in skeleton for a frontend/client-app spec
│   ├── FEATURE-TEMPLATE.md         ← fill-in skeleton for a new feature file
│   └── PROMPT-TEMPLATE.md          ← the dispatch-header + body shape every prompt follows
└── .claude/skills/                 ← the workflow's recurring motions, packaged as skills
    ├── generate-prompts/           ← Step 3: required reads, preconditions, prompt structure
    └── close-loop/                 ← Step 5: the full close-out checklist
```

## The workflow in one breath

1. **Design** a feature in a `Features/FEATURE-*.md` scratchpad — the agent
   drafts it from your intent against the relevant service specs; you refine.
   The file records the decisions, the **implementation order** across
   services, and a **recommended model tier per service** (cheapest that fits —
   decided at design time, not session time).
2. **Cascade** the agreed design into the affected service specs — written as
   if already built, marked `<!-- PENDING: FEATURE-{name} -->` — and add a
   Pending row to `STATUS.md`.
3. **Generate** one self-contained prompt per affected service (the
   `generate-prompts` skill). Every prompt opens with a **dispatch header** —
   target repo, branch, prerequisites, status
   (`Generated → Applied → Verified`), recommended model — which turns
   `Prompts/` into a dispatch board.
4. **Implement** each prompt in a fresh agent session inside its service repo,
   in the declared order; a prompt whose prerequisites aren't `Verified` must
   not be applied. Human verification flips the status.
5. **Close the loop** (the `close-loop` skill): verify the specs **against the
   code, not memory**, reconcile any drift (reality wins), remove the `PENDING`
   markers, flip the `STATUS.md` row, write one dated `CHANGELOG.md` entry with
   the implementing commit refs, archive the feature file and prompts, and sync
   `repo-instructions/` if a convention changed.

See [WORKFLOW.md](WORKFLOW.md) for the full version. The gift-cards feature —
one cross-service feature touching `main-api` and both frontends — is traced
through steps 1–3 in `Features/` and `Prompts/`.

## The rules that keep the hub honest

- **Specs describe what is implemented; feature files describe what is not** —
  and cascaded-but-unbuilt spec sections carry a `PENDING` marker in between.
- **Exactly one document accumulates history.** `CHANGELOG.md` takes dated
  entries; `STATUS.md` notes are one line, ever; every other file describes
  current state only.
- **Close-out verifies against the code**, with commit SHAs captured for
  traceability — a spec is only a source of truth if something regularly forces
  it to be true.
- **Every distinct kind of question gets the smallest file that can answer
  it** — conventions and status were extracted from the overview so an agent
  never loads the full architecture document just to check a naming rule. When
  a service spec outgrows the context window, the same idea applies one level
  down (the split-spec index pattern — see `SPEC-GUIDELINES.md`; Plaza's specs
  are still small enough not to need it).
- **Parked designs get their own state** (`Features/Staled/`) so they read
  neither as active nor as shipped, and must be re-validated before revival.

## Who authors what

In practice an agent drafts all of these files; the distinction that matters is
**ownership and durability**, not who typed them:

- **Service specs** — AI-drafted, **human-curated source of truth**. Durable.
- **Feature files** — AI-drafted from your intent, **human-refined scratchpad**.
  Archived (not deleted) after shipping: the contracts graduate into the spec,
  the design rationale survives in `Features/Implemented/`.
- **Prompts** — AI-generated, **human-reviewed**. Archived after the work lands.

The invariant: the agent never edits a spec *as part of building a feature*.
Every spec change is a deliberate, reviewed step (steps 2 and 5).
