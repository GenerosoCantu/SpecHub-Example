# Plaza Main API — Specification Document

> The NestJS backend-for-frontend. Owns products, orders, store configuration,
> and gift cards; issues auth tokens; publishes JSON snapshots to the asset
> service.

**Service identifier:** `main-api`
**Last updated:** 2026-06-24 — added Gift Cards module (schema, endpoints, checkout redemption).

---

## Table of Contents

1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Architecture](#architecture)
4. [Authentication](#authentication)
5. [Data Models](#data-models)
6. [Endpoints](#endpoints)
7. [Endpoint Summary](#endpoint-summary)
8. [Environment Variables](#environment-variables)
9. [Cross-Cutting Concerns / Patterns](#cross-cutting-concerns--patterns)
10. [Publish / Snapshot Flow](#publish--snapshot-flow)
11. [Key Design Notes](#key-design-notes)
12. [Known Issues & Gaps](#known-issues--gaps)

---

## Overview

- **Purpose:** the single backend the admin dashboard talks to, and the public
  cart/checkout API the storefront calls. It is the only writer of the store's
  published JSON snapshot.
- **Owns:** `Product`, `Order`, `GiftCard`, store configuration, and user auth.
- **Does not own:** the store/tenant record (that is `tenant-service`), or file
  storage (that is `asset-service`).

---

## Tech Stack

| Concern      | Choice                                |
| ------------ | ------------------------------------- |
| Language     | TypeScript                            |
| Framework    | NestJS 10                             |
| Datastore    | MongoDB Atlas — database `plaza` (Mongoose) |
| Default port | 4000                                  |
| Hosting      | VPS under pm2                         |

---

## Architecture

```text
main-api/
├── src/
│   ├── auth/            # login, refresh, /auth/validate
│   ├── products/        # product CRUD + publish trigger
│   ├── orders/          # cart, checkout, order lifecycle
│   ├── gift-cards/      # issue, list, redeem  ← added by gift-cards feature
│   ├── store-config/    # per-store settings, theme, domains
│   ├── publish/         # builds + writes JSON snapshot via asset-service
│   ├── common/          # guards, interceptors, filters, dto base
│   └── main.ts
```

### Global Middleware / Bootstrap

- `ValidationPipe` (whitelist + transform) registered globally.
- `GeneralInterceptor` attaches `x-request-id` and shapes list responses.
- `HttpExceptionFilter` produces the standard error envelope.
- CORS allows the admin and storefront origins only.

---

## Authentication

- **Strategy:** JWT bearer access tokens (15 min) + refresh tokens (7 days),
  both issued by this service on seller login.
- **Guard:** `JwtAuthGuard` on all protected routes; `RolesGuard` for
  role-restricted ones.
- **Claims:** `sub` (user id), `storeId` (tenant), `role`.
- **Delegated validation:** other services call `POST /auth/validate` with a
  bearer token; this service returns the decoded claims (or 401). Signing keys
  never leave `main-api`.
- **Tenant isolation:** every product/order/gift-card query is filtered by the
  `storeId` claim unconditionally.

---

## Data Models

### Product (collection `products`)

| Field       | Type     | Required | Default  | Notes / Index               |
| ----------- | -------- | -------- | -------- | --------------------------- |
| `_id`       | ObjectId | yes      | auto     | PK                          |
| `storeId`   | string   | yes      | —        | indexed; tenant isolation   |
| `slug`      | string   | yes      | —        | unique per store; public URL|
| `title`     | string   | yes      | —        |                             |
| `priceCents`| number   | yes      | —        | integer, minor units        |
| `active`    | boolean  | yes      | `true`   | hidden from storefront if false |

### Order (collection `orders`)

| Field          | Type     | Required | Default     | Notes                        |
| -------------- | -------- | -------- | ----------- | ---------------------------- |
| `_id`          | ObjectId | yes      | auto        | PK                           |
| `storeId`      | string   | yes      | —           | indexed                      |
| `items`        | array    | yes      | —           | `{ productId, qty, priceCents }` |
| `subtotalCents`| number   | yes      | —           |                              |
| `giftCardCode` | string   | no       | —           | applied gift card, if any    |
| `discountCents`| number   | yes      | `0`         | amount covered by gift card  |
| `totalCents`   | number   | yes      | —           | `subtotal - discount`        |
| `status`       | string   | yes      | `pending`   | `pending`/`paid`/`cancelled` |

### GiftCard (collection `gift_cards`)

| Field          | Type     | Required | Default   | Notes / Index                       |
| -------------- | -------- | -------- | --------- | ----------------------------------- |
| `_id`          | ObjectId | yes      | auto      | PK                                  |
| `storeId`      | string   | yes      | —         | indexed; tenant isolation           |
| `code`         | string   | yes      | —         | unique per store; 12 chars, uppercase |
| `initialCents` | number   | yes      | —         | issued value                        |
| `balanceCents` | number   | yes      | =initial  | decremented on redemption           |
| `active`       | boolean  | yes      | `true`    | deactivated cards cannot be redeemed |
| `createdAt`    | Date     | yes      | now       |                                     |

---

## Endpoints

### Public Endpoints (no auth — storefront)

#### `GET /public/products` — list active products for a store

- **Guard:** none; resolves store from the `x-store-domain` header.
- **Response:** `{ data: Product[] }` (active only).

#### `POST /public/checkout` — create an order

- **Guard:** none; store resolved from `x-store-domain`.
- **Request DTO:** `CheckoutDto` — `{ items: {productId, qty}[], giftCardCode?: string }`
- **Response:** `Order` with computed `discountCents` and `totalCents`.
- **Side effects:** if `giftCardCode` is present and valid, decrements the gift
  card balance atomically (see redemption rule below).

### Protected Endpoints (JWT required — admin)

#### `POST /products` — create a product

- **Guard:** `JwtAuthGuard`
- **Request DTO:** `CreateProductDto` — `{ title, slug, priceCents, active }`
- **Response:** `Product`. Triggers a snapshot publish.

#### `GET /gift-cards` — list a store's gift cards

- **Guard:** `JwtAuthGuard`
- **Response:** `{ data: GiftCard[] }` scoped by `storeId`.

#### `POST /gift-cards` — issue a gift card

- **Guard:** `JwtAuthGuard` + `RolesGuard('owner','manager')`
- **Request DTO:** `IssueGiftCardDto` — `{ initialCents: number }`
- **Response:** `GiftCard`. `code` is server-generated (12-char uppercase,
  unique per store); `balanceCents` initialized to `initialCents`.

#### `PATCH /gift-cards/:id/deactivate` — deactivate a gift card

- **Guard:** `JwtAuthGuard` + `RolesGuard('owner','manager')`
- **Response:** updated `GiftCard` with `active: false`.

---

## Endpoint Summary

| Method | Path                            | Auth        | Purpose                  |
| ------ | ------------------------------- | ----------- | ------------------------ |
| GET    | `/public/products`              | —           | List active products     |
| POST   | `/public/checkout`              | —           | Create order (+ redeem)  |
| POST   | `/products`                     | JWT         | Create product           |
| GET    | `/gift-cards`                   | JWT         | List gift cards          |
| POST   | `/gift-cards`                   | JWT + role  | Issue gift card          |
| PATCH  | `/gift-cards/:id/deactivate`    | JWT + role  | Deactivate gift card     |
| POST   | `/auth/validate`                | bearer      | Validate token (internal)|

---

## Environment Variables

| Variable             | Required | Default | Controls                          |
| -------------------- | -------- | ------- | --------------------------------- |
| `PORT`               | no       | 4000    | listen port                       |
| `MAIN_API_MONGO_URI` | yes      | —       | Atlas connection string           |
| `MAIN_API_JWT_SECRET`| yes      | —       | access/refresh token signing key  |
| `TENANT_SERVICE_URL` | yes      | —       | base URL for store lookups        |
| `ASSET_SERVICE_URL`  | yes      | —       | base URL for snapshot/image writes|

---

## Cross-Cutting Concerns / Patterns

- **Validation:** global `ValidationPipe`; DTOs use `class-validator`.
- **Response envelope:** single resource returned bare; lists as `{ data, meta }`.
- **Errors:** `HttpExceptionFilter` → `{ statusCode, message, requestId }`.
- **Tenant scoping:** a `@CurrentStore()` decorator extracts `storeId` from the
  JWT; repositories require it on every read/write.

---

## Publish / Snapshot Flow

```text
admin writes (product/config change)
        │
        ▼
publish module builds snapshot JSON for the store
        │  PATCH {ASSET_SERVICE_URL}/files/{storeId}/snapshot
        ▼
asset-service writes stores/{storeId}/snapshot.json
        │
        ▼
storefront reads the static snapshot (no live DB hit)
```

Gift-card balances are **not** written to the snapshot (they change at checkout
and would stale the static file); redemption always validates live via
`POST /public/checkout`.

---

## Key Design Notes

- **Gift-card redemption is atomic:** a redemption uses a single
  `findOneAndUpdate({ code, storeId, active: true, balanceCents: { $gte: amount } }, { $inc: { balanceCents: -amount } })`
  so two concurrent checkouts cannot overspend a card.
- **Discount cannot exceed subtotal:** applied discount is
  `min(balanceCents, subtotalCents)`; remaining balance stays on the card.
- **Codes are per-store unique**, not globally unique — the index is
  `{ storeId, code }`.

---

## Known Issues & Gaps

- No partial-refund path back onto a gift card yet.
- Deactivating a card does not notify the buyer who holds it.
- `[PENDING]` storefront balance-check endpoint (show remaining balance before
  checkout) — see Open Questions #1 in the overview.
