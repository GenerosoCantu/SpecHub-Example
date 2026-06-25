<!--
  FRONTEND SPEC TEMPLATE
  ======================
  Copy this file to `##-{frontend}.md` at the workspace root and fill it in.
  The numeric prefix sets reading order in the Spec Document Index.

  Use THIS template for a client app (storefront, dashboard, control center).
  Use SERVICE-SPEC-TEMPLATE.md for a backend service (API, worker, gateway).

  This is the fill-in skeleton. For the rules on WHAT each section must
  contain, read SPEC-GUIDELINES.md §2 ("Service Specs") and the
  service-specific expectations for frontends — that is the canonical
  checklist; this template is just the scaffold to populate.

  Two common flavors of frontend, and the sections each one keeps:
    A. Snapshot / SSR public site (e.g. storefront-frontend, Next.js):
       reads a static published JSON snapshot for most content and only
       calls the live API for a few actions. KEEP: Snapshot Consumption,
       Live API Usage. DROP: Redux Store Structure (or keep minimal).
    B. Authenticated SPA dashboard (e.g. admin-frontend, React + Redux):
       talks only to an authenticated API. KEEP: Base Request Layer,
       Redux Store Structure, API Services. DROP: Snapshot Consumption.
  Keep the sections that match your app; delete the rest. Do not pad
  sections that genuinely do not apply.

  How to generate one with an agent:
    1. Tell the agent which frontend repo it is, and point it at the repo.
    2. Have it read SPEC-GUIDELINES.md §2 and this template.
    3. Ask it to fill every section from the actual source — exact slice
       names, real route paths, real API service file names, real env vars.
       No speculation.
    4. Anything unresolved goes in "Known Issues & Gaps", not inline guesses.

  Rules of thumb (from SPEC-GUIDELINES.md):
    - Describe the IMPLEMENTED state, not the idea or a design draft.
    - Use exact identifiers (slice names, route paths, component/file names).
    - Prefer tables over prose; show real request shapes where it helps.
    - A frontend OWNS UI state only — never claim it owns domain data.
    - Delete this comment block and every <!-- guidance --> note before committing.
    - Remove sections that genuinely do not apply; do not pad them.
-->

# Plaza {Frontend Name} — Specification Document

> One- or two-sentence description: who this app is for, what it manages or
> renders, and which backend/data source it talks to.

**Service identifier:** `{frontend-id}`  <!-- e.g. storefront-frontend, admin-frontend -->
**Last updated:** {YYYY-MM-DD} — {brief note on the most recent change}

---

## Table of Contents

1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Architecture](#architecture)
4. [Authentication & Session](#authentication--session)
5. [Base Request Layer](#base-request-layer)        <!-- flavor B -->
6. [State / Redux Store Structure](#state--redux-store-structure)  <!-- flavor B -->
7. [API Services](#api-services)                     <!-- flavor B -->
8. [Snapshot Consumption](#snapshot-consumption)     <!-- flavor A -->
9. [Live API Usage](#live-api-usage)                 <!-- flavor A -->
10. [Routes, Views & Pages](#routes-views--pages)
11. [Enumerations & Constants](#enumerations--constants)
12. [Environment Variables](#environment-variables)
13. [Frontend Patterns & Conventions](#frontend-patterns--conventions)
14. [Key Design Notes](#key-design-notes)
15. [Known Issues & Gaps](#known-issues--gaps)

<!-- Renumber and drop entries to match the sections you actually keep. -->

---

## Overview

- **Purpose:** what this app lets its user do, and for whom (buyer, seller, admin).
- **Owns:** UI/presentation state only. Name the slices or local stores it owns.
- **Does not own:** domain data — name the service that does (link its spec).
- **Multi-tenancy:** how the current tenant/store is determined (domain lookup,
  a claim in the token, a path segment) and how that scopes what the user sees.

---

## Tech Stack

| Concern        | Choice                                       |
| -------------- | -------------------------------------------- |
| Framework      | {e.g. React + Redux / Next.js (ISR)}         |
| Language       | {e.g. TypeScript}                            |
| Styling / UI   | {e.g. Tailwind, MUI, CSS modules}            |
| Data fetching  | {e.g. RTK Query / thunks / Next data fetching}|
| Build / output | {e.g. static SPA / SSR + ISR}                |
| Default port   | {e.g. 3000}                                  |
| Hosting        | {e.g. CDN / Vercel / VPS}                     |

---

## Architecture

How the app is laid out and where data comes from. A folder tree, a render-flow
diagram, or both.

```text
{frontend-id}/
├── src/
│   ├── app|pages/        # routes / pages
│   ├── components/       # shared presentational components
│   ├── features/         # feature folders (view + slice + api)
│   │   └── {feature}/
│   │       ├── {Feature}View.tsx
│   │       ├── {feature}Slice.ts
│   │       └── {feature}Api.ts
│   ├── store/            # store config, root reducer  (flavor B)
│   ├── services/         # apiService + per-resource API services (flavor B)
│   └── lib/              # helpers, constants, formatters
```

```text
# Render / data-flow (adapt to your flavor)
domain or login → resolve tenant → fetch data → render
                                       │
       flavor A: static snapshot   |   flavor B: authenticated API calls
```

---

## Authentication & Session

- **Who logs in (if anyone):** {seller / admin / nobody — public}.
- **Token handling:** where the access/refresh token is stored, how it is
  attached to requests, and how the tenant claim (`storeId`) is read.
- **Session lifecycle:** login, refresh-on-401, logout/expiry behavior.
- **Public surface:** which routes/pages render without a session.

<!-- For a fully public, snapshot-only frontend this may be a single line:
     "No user auth; the app is read-only and resolves the store by domain." -->

---

## Base Request Layer
<!-- FLAVOR B (authenticated SPA). Delete for a public snapshot site. -->

- **Entry point:** `{apiService.makeRequest(method, path, body)}` — what it does
  for every call (attaches base URL + bearer token, sets headers, parses the
  response envelope, routes errors through one handler).
- **Tenant resolution:** how the base URL or tenant header is derived
  (`{tenantUrl}` / `x-store-domain` / claim-based).
- **Auth retry:** behavior on `401` (e.g. attempt refresh once, then retry or
  redirect to login).
- **Error surface:** how API errors reach the UI (toast, error slice, thrown).

---

## State / Redux Store Structure
<!-- FLAVOR B. For flavor A, replace with a short note on local/UI state only. -->

One row per top-level slice. Use the real slice names.

| Slice         | Holds                                            |
| ------------- | ------------------------------------------------ |
| `auth`        | session, tokens, current user + `storeId`        |
| `{resource}`  | the list + an edit buffer for the create/edit form|
| `ui`          | modals, toasts, global loading flags             |

- **Async pattern:** {thunks / RTK Query / sagas} — one line on how requests
  flow into slices.
- **Edit-buffer convention:** how an in-progress create/edit is held separately
  from the canonical list (see Frontend Patterns).

---

## API Services
<!-- FLAVOR B. One row per resource module that wraps backend endpoints. -->

| Service file   | Wraps (backend endpoints)                  |
| -------------- | ------------------------------------------ |
| `{resource}Api`| `/{resource}` CRUD                         |
| `{resource}Api`| `/{resource}`, `/{resource}/:id/{action}`  |

Show the method-to-endpoint mapping for any non-obvious resource:

| Method   | Call                              | Endpoint                       |
| -------- | --------------------------------- | ------------------------------ |
| list     | `{resource}Api.list()`            | `GET /{resource}`              |
| create   | `{resource}Api.create(dto)`       | `POST /{resource}`             |
| action   | `{resource}Api.{action}(id)`      | `PATCH /{resource}/:id/{action}`|

---

## Snapshot Consumption
<!-- FLAVOR A (snapshot / SSR public site). Delete for an SPA dashboard. -->

- **Snapshot path:** `{stores/{storeId}/snapshot.json}`, served from `{CDN}`.
- **Contains:** {store config, theme, active catalog/content} — what is baked in.
- **Refresh model:** {ISR revalidation interval / on-publish webhook}. The app
  never queries the database directly.
- **Static vs live:** be explicit about what renders from the snapshot and what
  must hit the live API (and why — e.g. values that go stale, like balances).

---

## Live API Usage
<!-- FLAVOR A. The few live calls a snapshot site still makes. -->

| Action          | Endpoint                          | Why live (not snapshot)        |
| --------------- | --------------------------------- | ------------------------------ |
| {List products} | (from snapshot)                   | —                              |
| {Checkout}      | `POST {service} /public/checkout` | needs live pricing / inventory |

---

## Routes, Views & Pages

One row per route. Note the data source (snapshot vs API) and auth requirement.

| Route                    | View / Page          | Data source        | Auth |
| ------------------------ | -------------------- | ------------------ | ---- |
| `/`                      | {Home}               | {snapshot / API}   | {—/✓}|
| `/{resource}`            | {List + create/edit} | `{resource}Api`    | ✓    |
| `/{resource}/[id]`       | {Detail}             | {snapshot / API}   | {—/✓}|

For each non-trivial view, add a short paragraph: what it shows, the key actions
the user can take, and which API calls or slices back them.

---

## Enumerations & Constants

App-level enums, status values, and magic constants the UI relies on. Keep them
in sync with the backend spec that defines them (link it).

| Name              | Values / Definition                  | Used by                |
| ----------------- | ------------------------------------ | ---------------------- |
| `{OrderStatus}`   | `pending` \| `paid` \| `cancelled`   | order list badge       |
| `{ROUTES}`        | central route-path constants         | navigation             |

---

## Environment Variables

| Variable             | Required | Default | Controls                        |
| -------------------- | -------- | ------- | ------------------------------- |
| `{API_BASE_URL}`     | yes      | —       | base URL for the backend/API    |
| `{CDN_BASE_URL}`     | {y/n}    | —       | where snapshots are read from   |
| `PORT`               | no       | {value} | dev server / listen port        |

<!-- Note the build-time vs runtime distinction if it matters
     (e.g. NEXT_PUBLIC_* are inlined at build). -->

---

## Frontend Patterns & Conventions

The repeatable patterns a contributor must follow to stay consistent.

- **Create/edit view pattern:** {a shared form component backed by an edit-buffer
  slice; submit dispatches the API thunk and refreshes the list}.
- **List + detail pattern:** how lists load, paginate, and link to detail.
- **Loading & error UX:** standard spinner/skeleton and error placement.
- **Forms & validation:** library and where client validation lives vs.
  server-only validation.
- **Optimistic vs. refetch:** the default after a mutation.
- **Feature-folder convention:** how a new feature's view, slice, and API service
  are colocated and wired in.

---

## Key Design Notes

Decisions worth remembering: why content is static vs. live, why a value is never
cached in the snapshot, refresh/revalidation choices, tenant-resolution strategy,
any deliberate "the frontend does not do X — the API does" boundaries.

---

## Known Issues & Gaps

- {Known bug, missing affordance, or UX debt.}
- {Anything a `[PENDING]` feature still depends on — link the overview's Open
  Questions entry.}

<!-- Speculative / unresolved items live here, never inline as if implemented. -->
