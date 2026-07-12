# Plaza — Shared Conventions

> **Source of truth for cross-service conventions.** Extracted from `00-architecture-overview.md` Section 8 so an agent can check a naming rule without loading the full overview. Read this file for API naming, domain entities, storage layout, env vars, logging, and code organization. System-wide questions (components, tenancy, auth, communication) still belong to `00-architecture-overview.md`.

These apply to **all** services and are what stop convention drift — every generated prompt inherits them.

## API Naming

- REST, plural nouns, kebab paths (`/gift-cards`), JSON bodies.
- Public (storefront-facing) endpoints live under `/public/*` and resolve the store from the `x-store-domain` header; everything else requires a JWT.

## Domain Entities

| Entity     | Owner        | ID strategy    | Notes                                    |
| ---------- | ------------ | -------------- | ---------------------------------------- |
| `Store`    | tenant-service | ObjectId     | tenant record; domains + `active` flag   |
| `User`     | main-api     | ObjectId       | seller accounts; `role` claim            |
| `Product`  | main-api     | ObjectId       | public `slug`, unique per store          |
| `Order`    | main-api     | ObjectId       | minor-unit `*Cents` money fields         |
| `GiftCard` | main-api     | ObjectId       | `code` unique per store (`{storeId, code}`) |

- **Money:** always integer minor units, field names suffixed `Cents`.
- **Tenant isolation:** every domain document carries `storeId`; every query is scoped by it unconditionally.

## Snapshot / Storage Model

- Published JSON lives at `stores/{storeId}/snapshot.json`; images at `stores/{storeId}/assets/...`.
- Only `main-api` writes snapshots (via `asset-service`); the storefront only reads them.
- Data that changes at request time (e.g. gift-card balances) is **never** written to a snapshot — it is validated live.

## Environment Variables

- Prefixed per service: `MAIN_API_*`, `TENANT_*`, `ASSET_*`.
- `PORT` is the only unprefixed variable each service reads.

## Logging

- Structured JSON logs; request id propagated via `x-request-id`.

## Code Organization

- Backend services: one folder per feature module (`{feature}.module.ts`, `{feature}.service.ts`, `{feature}.controller.ts`, `{feature}.schema.ts`, plus `dto/`).
- Admin frontend: feature-based Redux slices plus one API service file per resource.

## IDs

- Mongo `ObjectId` internally; human-facing slugs where a spec requires them.
