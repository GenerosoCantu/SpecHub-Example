# Plaza Admin Frontend — Specification Document

> The seller dashboard. A React + Redux SPA that manages one store's products,
> orders, configuration, and gift cards via the authenticated `main-api`.

**Service identifier:** `admin-frontend`
**Last updated:** 2026-06-24 — added Gift Cards management view.

---

## Table of Contents

1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Base Request Layer](#base-request-layer)
4. [Redux Store Structure](#redux-store-structure)
5. [API Services](#api-services)
6. [Views & Pages](#views--pages)
7. [Frontend Patterns](#frontend-patterns)
8. [Known Issues & Gaps](#known-issues--gaps)

---

## Overview

- **Purpose:** let a seller manage their store.
- **Owns:** UI state only; all domain data lives in `main-api`.
- **Auth:** seller logs in; the access token (carrying `storeId`) authorizes
  every request.

## Tech Stack

| Concern    | Choice           |
| ---------- | ---------------- |
| Framework  | React + Redux    |
| Port       | 3100             |
| Build      | Static SPA       |

## Base Request Layer

- `apiService.makeRequest(method, path, body)` attaches the bearer token and
  base URL, and routes errors through a shared handler.
- A 401 triggers a refresh-token attempt, then retry once.

## Redux Store Structure

| Slice       | Holds                                  |
| ----------- | -------------------------------------- |
| `auth`      | session, tokens, current user + storeId |
| `products`  | product list + edit buffer             |
| `orders`    | order list + filters                   |
| `giftCards` | gift card list + issue/deactivate state |

## API Services

| Service file          | Wraps                                  |
| --------------------- | -------------------------------------- |
| `productsApi`         | `/products` CRUD                       |
| `ordersApi`           | `/orders` read                         |
| `giftCardsApi`        | `/gift-cards`, `/gift-cards/:id/deactivate` |

### Gift Cards API (added by gift-cards feature)

| Method | Call                              | Endpoint                          |
| ------ | --------------------------------- | --------------------------------- |
| list   | `giftCardsApi.list()`             | `GET /gift-cards`                 |
| issue  | `giftCardsApi.issue({initialCents})` | `POST /gift-cards`             |
| off    | `giftCardsApi.deactivate(id)`     | `PATCH /gift-cards/:id/deactivate`|

## Views & Pages

- `/products` — list/create/edit products
- `/orders` — order list, shows applied gift-card discount per order
- `/gift-cards` — **(new)** list cards (code, initial, balance, status); issue a
  card by amount; deactivate a card

## Frontend Patterns

- **Create/edit view pattern:** a shared form component backed by an edit buffer
  slice; submit dispatches the API thunk and refreshes the list.
- Gift Cards view follows this pattern: an "Issue gift card" modal (amount only;
  code is server-generated) plus a table with a per-row Deactivate action.

## Known Issues & Gaps

- No CSV export of gift cards yet.
