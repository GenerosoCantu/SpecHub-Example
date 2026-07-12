# PROMPT — admin-frontend — Gift Cards

> **Target repo:** plaza-admin-frontend
> **Branch:** feature/gift-cards
> **Prerequisites:** PROMPT-main-api-gift-cards.md (must be Verified)
> **Status:** Generated   <!-- Generated → Applied → Verified -->
> **Recommended model:** Sonnet — standard list/modal view, but spans a new slice + API service + view wiring

<!--
  Self-contained implementation prompt, generated from 02-admin-frontend.md and
  Features/FEATURE-gift-cards.md (Step 3). Do not apply until the prerequisite
  main-api prompt is Verified — the endpoints below must exist.
-->

---

## Context

You are working in the `plaza-admin-frontend` repository (React + Redux SPA, the
Plaza seller dashboard). The backend already exposes gift-card endpoints on
`main-api`. Add a Gift Cards management view: sellers list their store's cards,
issue a new card by amount, and deactivate a card. All requests carry the
seller's JWT; the backend scopes everything by `storeId`.

**Pattern reference:** study the `products` feature — its slice, its
`productsApi` service file, and the `/products` list view. Gift Cards follows
the same create/edit view pattern: a shared form component backed by an edit
buffer slice; submit dispatches the API thunk and refreshes the list. Also study
`apiService.makeRequest(method, path, body)` — every call goes through it.

## Task

### 1. API service — `giftCardsApi`

| Method | Call                                 | Endpoint                           |
| ------ | ------------------------------------ | ---------------------------------- |
| list   | `giftCardsApi.list()`                | `GET /gift-cards`                  |
| issue  | `giftCardsApi.issue({initialCents})` | `POST /gift-cards`                 |
| off    | `giftCardsApi.deactivate(id)`        | `PATCH /gift-cards/:id/deactivate` |

- `list` resolves `{ data: GiftCard[] }`; `issue` and `deactivate` resolve the
  single `GiftCard`.

### 2. Redux slice — `giftCards`

- State: card list + issue/deactivate request state (loading/error), mirroring
  the `products` slice shape.
- Thunks wrap the three `giftCardsApi` calls; a successful issue or deactivate
  refreshes the list.

### 3. View — `/gift-cards`

- Table columns: **code**, **initial** (`initialCents`), **balance**
  (`balanceCents`), **status** (`active`).
- "Issue gift card" opens a modal with a single amount field (minor units;
  positive integer). The code is server-generated — the modal must not ask for
  one. On success, show the new card's code.
- Per-row **Deactivate** action with a confirm step; a deactivated row shows
  status inactive.
- Add the route and navigation entry alongside the existing `/products` and
  `/orders` entries.

### 4. Orders view

- On `/orders`, show the applied gift-card discount per order
  (`discountCents`, present on the `Order` returned by the API; `0` when no
  card was applied).

## UX states

- Loading: table skeleton/spinner consistent with the products list.
- Empty: "No gift cards yet" plus the issue button.
- Error: route API errors through the shared error handler (401 refresh/retry
  already handled by `apiService`).
- Permission: issuing/deactivating is owner/manager-only server-side; surface
  the API's 403 message rather than hiding the buttons in v1.

## Constraints

- Money is integer minor units end-to-end — never send or store floats.
- Do not add any client-side balance math beyond displaying returned fields.

## Acceptance

- A seller can issue a card and immediately see it listed with
  `balance == initial`.
- Deactivating a card flips its status in the list without a full page reload.
- An order paid partly with a gift card shows its discount on `/orders`.
