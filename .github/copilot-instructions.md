<!--
  EXAMPLE per-service-repo routing file.

  This file does NOT live in the Spec Hub in real use — it lives at the root of
  each *service* repository (e.g. the main-api repo). It is included here only so
  the example is self-contained and you can see what it looks like.

  Equivalent file names for other agents: CLAUDE.md (Claude Code), AGENTS.md (Codex).

  Its only job: route a fresh agent session to the right spec BEFORE it touches
  code. It holds stable, repo-wide conventions and nothing feature-specific, so
  there is almost nothing here that can drift from the hub.
-->

# main-api — Agent Instructions

You are working in the **`main-api`** repository of the Plaza platform: the
NestJS backend-for-frontend that owns products, orders, store configuration, and
gift cards, and publishes JSON snapshots to the asset service.

## Before doing anything

The authoritative design for this service lives in the **Spec Hub**, not in this
repo. For any non-trivial task:

1. Read the service spec: `03-main-api.md` in the Spec Hub.
2. If the task is system-wide (auth, multi-tenancy, cross-service flow), also read
   `00-architecture-overview.md`.
3. Implement strictly from the prompt you were given
   (`Prompts/PROMPT-main-api-*.md`). The prompt is self-contained — do not invent
   contracts that are not in it or the spec.

Never edit the specs from inside this repo. Spec changes happen in the hub as a
deliberate, reviewed step.

## Stable conventions (these rarely change)

- **Stack:** NestJS 10, TypeScript, MongoDB (Mongoose), Atlas database `plaza`.
- **Module layout:** one folder per feature — `{feature}.module.ts`,
  `{feature}.service.ts`, `{feature}.controller.ts`, `{feature}.schema.ts`,
  plus `dto/`.
- **API naming:** REST, plural nouns (`/products`, `/orders`, `/gift-cards`).
- **Auth:** JWT bearer; `JwtAuthGuard` on protected routes. The `storeId` claim
  enforces tenant isolation — every query is scoped by `storeId`.
- **Responses:** success returns the resource or `{ data, meta }` for lists;
  errors use the global `HttpExceptionFilter` envelope.
- **IDs:** Mongo `ObjectId`; public-facing slugs where the spec calls for them.

When in doubt about a contract, the spec wins. When the spec is silent, ask
rather than guess.
