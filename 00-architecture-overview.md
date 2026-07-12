# Plaza — Architecture Overview

**Version:** 1.1
**Last Updated:** 2026-07-11

> This documents the Plaza system at a high level and is the first file an
> agent reads for any **system-wide** question. Narrower questions have
> smaller files: shared conventions live in `CONVENTIONS.md`, feature status
> in `STATUS.md`, implementation history in `CHANGELOG.md` — an agent should
> load those instead of this overview when that is all it needs. The header
> above carries a version and a single date, nothing more; dated history
> belongs only in `CHANGELOG.md` (single-log rule, `WORKFLOW.md` Step 5).

## Table of Contents

1. [Product Summary](#1-product-summary)
2. [System Components](#2-system-components)
3. [High-Level Architecture Diagram](#3-high-level-architecture-diagram)
4. [Multi-Tenancy Strategy](#4-multi-tenancy-strategy)
5. [Authentication and Authorization](#5-authentication-and-authorization)
6. [Service Communication](#6-service-communication)
7. [Hosting and Deployment](#7-hosting-and-deployment)
8. [Shared Conventions](#8-shared-conventions)
9. [Spec Document Index](#9-spec-document-index)
10. [Known Gaps and Technical Debt](#10-known-gaps-and-technical-debt)
11. [Open Questions Log](#11-open-questions-log)
12. [Feature Status](#12-feature-status)

---

## 1. Product Summary

Plaza is a **multi-tenant storefront platform**. Each tenant is a *store* with
its own public website, product catalog, and orders. Sellers manage their store
through an admin dashboard; buyers browse and check out on a fast, mostly-static
public storefront. The platform's job is to let one codebase serve many isolated
stores cheaply, with published content delivered as static JSON.

---

## 2. System Components

### Client applications

| Component             | Stack    | Port | Purpose                                            |
| --------------------- | -------- | ---- | -------------------------------------------------- |
| `storefront-frontend` | Next.js  | 3000 | Public buyer-facing site, one per store, by domain |
| `admin-frontend`      | React+Redux | 3100 | Seller dashboard for managing a store           |

### Backend services

| Component        | Stack  | Port | Purpose                                                    |
| ---------------- | ------ | ---- | ---------------------------------------------------------- |
| `main-api`       | NestJS | 4000 | Backend-for-frontend; owns products, orders, gift cards    |
| `tenant-service` | NestJS | 4010 | Resolves a store by domain; owns the `stores` collection   |
| `asset-service`  | NestJS | 4020 | Image uploads + published JSON snapshots (the CDN writer)  |

### Data / infrastructure

| Component      | Purpose                                              |
| -------------- | ---------------------------------------------------- |
| MongoDB Atlas  | Shared cluster, database `plaza`; banners DB separate |
| Object storage | Holds uploaded images and published JSON snapshots   |

---

## 3. High-Level Architecture Diagram

```text
        buyers                          sellers
          │                                │
          ▼                                ▼
 ┌───────────────────┐          ┌────────────────────┐
 │ storefront-front. │          │   admin-frontend   │
 │   (Next.js)       │          │   (React+Redux)    │
 └─────────┬─────────┘          └──────────┬─────────┘
           │ static JSON + live API         │ live API
           │                                │
           ▼                                ▼
   ┌───────────────┐               ┌──────────────────┐
   │ asset-service │◄──publishes───│     main-api     │
   │  (JSON+images)│   snapshots   │  (BFF, NestJS)   │
   └───────────────┘               └───┬──────────┬───┘
                                       │          │
                         /auth/validate│          │ store lookup
                                       ▼          ▼
                              ┌─────────────────────────┐
                              │      tenant-service      │
                              └─────────────┬───────────┘
                                            │
                                            ▼
                                   ┌─────────────────┐
                                   │  MongoDB Atlas  │
                                   └─────────────────┘
```

---

## 4. Multi-Tenancy Strategy

- **Tenant = store.** Every store has an `_id` (`storeId`), one or more domains,
  and an `active` flag.
- **Resolution:** `tenant-service` maps an incoming domain/subdomain to a
  `storeId`. The storefront resolves its store at request time; the admin app
  carries `storeId` in the authenticated session.
- **Isolation:** every domain document carries `storeId`; every query in
  `main-api` is scoped by the `storeId` claim on the JWT. No cross-store reads.
- **Static delivery:** when a seller publishes, `main-api` writes a JSON snapshot
  of the store's catalog/config to the asset service. The storefront reads that
  snapshot statically and only calls the live API for cart and checkout.

---

## 5. Authentication and Authorization

- **Token issuer:** `main-api` issues short-lived JWT access tokens + refresh
  tokens on seller login.
- **Claims:** `sub` (user id), `storeId` (tenant), `role`.
- **Validation by other services:** `tenant-service` and `asset-service` do not
  decode tokens themselves; they delegate to `main-api` `POST /auth/validate`,
  which returns the resolved claims. This keeps signing keys in one place.
- **Tenant isolation:** the `storeId` claim is the boundary; protected queries
  filter by it unconditionally.

---

## 6. Service Communication

- `admin-frontend` → `main-api` (authenticated REST).
- `storefront-frontend` → asset snapshots (static) + `main-api` public endpoints
  (cart/checkout).
- `main-api` → `tenant-service` (store lookup), `main-api` → `asset-service`
  (publish JSON, upload images).
- `tenant-service` / `asset-service` → `main-api` `/auth/validate` (auth only).
- All inter-service calls are HTTP/JSON over the private network.

---

## 7. Hosting and Deployment

- Backend services (`main-api`, `tenant-service`, `asset-service`) run under pm2
  on a VPS.
- `storefront-frontend` deploys to a Node host with on-demand ISR.
- `admin-frontend` is a static SPA build served from object storage + CDN.
- MongoDB is hosted on Atlas. Published JSON + images live in object storage
  fronted by a CDN.

---

## 8. Shared Conventions

**Moved to [`CONVENTIONS.md`](CONVENTIONS.md).** The cross-service conventions
(API naming, domain entities, snapshot/storage model, env vars, logging, code
organization, IDs) were extracted into their own file so an agent checking a
naming rule doesn't have to load this whole overview. Every generated prompt
inherits them from there.

---

## 9. Spec Document Index

| File                          | Covers                                             |
| ----------------------------- | -------------------------------------------------- |
| `01-storefront-frontend.md`   | Public storefront: snapshot consumption, checkout  |
| `02-admin-frontend.md`        | Seller dashboard: Redux, API services, views       |
| `03-main-api.md`              | BFF: modules, auth, every endpoint, publish flow   |
| `04-tenant-service.md`        | Store resolution and the `stores` collection       |
| `05-asset-service.md`         | Image upload + JSON snapshot persistence            |

---

## 10. Known Gaps and Technical Debt

- Refresh-token rotation is issued but not yet revocable mid-session.
- The storefront re-fetches the full snapshot on any change; no partial updates.
- No soft-delete on `Product`; deletes are hard.

---

## 11. Open Questions Log

| # | Question                                                        | Blocking? |
| - | --------------------------------------------------------------- | --------- |
| 1 | Should gift-card balances be cached on the storefront snapshot? | No        |
| 2 | Do we need per-store currency, or is single-currency fine v1?   | No        |

---

## 12. Feature Status

**Moved to [`STATUS.md`](STATUS.md).** The live feature status board — one row
per feature, flipped to ✅ in step 5 — was extracted into its own file so an
agent checking a feature's state doesn't have to load this whole overview.
Implementation history lives in `CHANGELOG.md`.
