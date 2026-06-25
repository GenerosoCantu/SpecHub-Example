# Plaza Spec Hub — a worked example of the multi-service spec workflow

This repository is the **companion example** to the article
*"Shipping cross-repo features with an AI agent: a spec-first workflow."*
It is a small, **fictional** multi-tenant storefront platform ("Plaza") used
only to demonstrate the workflow — none of it is a real product.

The point of the repo is not the product. It is the **structure**: how specs,
feature files, prompts, and a routing file fit together so an AI agent can ship
a feature that spans several services without drifting or losing context.

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
would be separate repositories containing only source code plus a small routing
file (`.github/copilot-instructions.md`). This hub holds the specs that describe
them.

## What's in here

```text
.
├── 00-architecture-overview.md     ← the map; first stop for any system question
├── 01-storefront-frontend.md       ← service specs (one per service)
├── 02-admin-frontend.md
├── 03-main-api.md                  ← the fullest example spec
├── 04-tenant-service.md
├── 05-asset-service.md
├── SPEC-GUIDELINES.md              ← the canonical "what goes in a spec" checklist
├── WORKFLOW.md                     ← the feature lifecycle (the 5 steps)
├── templates/
│   ├── SERVICE-SPEC-TEMPLATE.md    ← fill-in skeleton for a new service spec
│   └── FEATURE-TEMPLATE.md         ← fill-in skeleton for a new feature file
├── Features/
│   ├── FEATURE-gift-cards.md       ← a worked, cross-service design scratchpad
│   └── Implemented/                ← archived feature files for shipped work
├── Prompts/
│   ├── PROMPT-main-api-gift-cards.md   ← a prompt generated from the spec
│   └── Implemented/                ← prompts for completed work
└── .github/
    └── copilot-instructions.md     ← example of the per-service-repo routing file
```

## The workflow in one breath

1. **Design** a feature in a `Features/FEATURE-*.md` scratchpad — the agent reads
   the relevant service specs to draft it; you refine.
2. **Cascade** the agreed design into the affected service specs and add a row to
   the Pending Features table in the overview.
3. **Generate** one self-contained prompt per affected service from those specs.
4. **Implement** each service in its own repo, in a fresh agent session, from its
   prompt.
5. **Close the loop:** update the specs to match what shipped, flip the feature to
   complete, and archive the feature file and prompts.

See [WORKFLOW.md](WORKFLOW.md) for the full version. The gift-cards feature in
`Features/` and `Prompts/` shows steps 1–3 end to end.

## Who authors what

In practice an agent drafts all of these files; the distinction that matters is
**ownership and durability**, not who typed them:

- **Service specs** — AI-drafted, **human-curated source of truth**. Durable.
- **Feature files** — AI-drafted from your intent, **human-refined scratchpad**.
  Disposable; archived after shipping.
- **Prompts** — AI-generated, **human-reviewed, disposable**. Discarded after the
  work lands.

The invariant: the agent never edits a spec *as part of building a feature*.
Every spec change is a deliberate, reviewed step (steps 2 and 5).
