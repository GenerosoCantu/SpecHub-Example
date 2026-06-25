# Plaza Storefront Frontend — Specification Document

> The public, buyer-facing storefront. One logical app serves every store,
> resolved by domain. Reads a static JSON snapshot for catalog/content and only
> calls the live API for cart and checkout.

**Service identifier:** `storefront-frontend`
**Last updated:** 2026-06-24 — added gift-card redemption field at checkout.

---

## Table of Contents

1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Architecture](#architecture)
4. [Snapshot Consumption](#snapshot-consumption)
5. [Live API Usage](#live-api-usage)
6. [Views & Pages](#views--pages)
7. [Key Design Notes](#key-design-notes)
8. [Known Issues & Gaps](#known-issues--gaps)

---

## Overview

- **Purpose:** render a fast, mostly-static store for buyers; handle cart and
  checkout against the live API.
- **Owns:** the public rendering and buyer cart state. Owns no domain data.
- **Multi-tenancy:** the incoming domain identifies the store; the matching
  snapshot is fetched and rendered.

---

## Tech Stack

| Concern   | Choice                  |
| --------- | ----------------------- |
| Framework | Next.js (ISR)           |
| Port      | 3000                    |
| Data in   | Static JSON snapshot + `main-api` public endpoints |

---

## Architecture

```text
domain → resolve store → fetch stores/{storeId}/snapshot.json (static)
                                   │
        catalog/content pages render from snapshot
                                   │
        cart/checkout → main-api /public/* (live)
```

## Snapshot Consumption

- Snapshot path: `stores/{storeId}/snapshot.json`, served from the CDN.
- Contains: store config, theme, and the active product catalog.
- Re-fetched on ISR revalidation; the storefront never queries Mongo directly.

## Live API Usage

| Action          | Endpoint                       |
| --------------- | ------------------------------ |
| List products   | (from snapshot)                |
| Checkout        | `POST main-api /public/checkout` |

### Gift-card redemption (added by gift-cards feature)

- The checkout form exposes an optional **gift card code** field.
- On submit, the entered code is sent as `giftCardCode` in `CheckoutDto`.
- The returned `Order` includes `discountCents` and `totalCents`; the UI shows
  the discount line and the amount still due.
- Validation is server-side only — the storefront does not check balances
  itself (balances are never in the static snapshot).

## Views & Pages

- `/` — store home (from snapshot)
- `/products/[slug]` — product detail (from snapshot)
- `/cart` — client cart state
- `/checkout` — checkout form incl. gift-card code field → `/public/checkout`

## Key Design Notes

- Catalog is static; only cart/checkout touch the live API. This keeps the
  storefront fast and cheap per store.

## Known Issues & Gaps

- No "check gift card balance before checkout" affordance yet (Open Questions #1).
