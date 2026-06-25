# FEATURE: Gift Cards

**Status:** In progress (specs cascaded; prompts generated)
**Affected services:** `main-api`, `admin-frontend`, `storefront-frontend`
**Not affected:** `tenant-service`, `asset-service`

> This is a design scratchpad, written before implementation and archived to
> `Features/Implemented/` once the feature ships. It is AI-drafted from the
> author's intent (the agent read the relevant service specs to draft it) and
> human-refined. It is *not* a spec — the agreed decisions below are cascaded
> into the service specs in step 2, and this file is the record of *why*.

---

## 1. Intent (what the author asked for)

> "Let sellers issue gift cards for their store. Buyers can enter a gift card
> code at checkout and have its balance applied to their order. Sellers can see
> all their cards and deactivate one."

## 2. Decisions

- **Ownership:** gift cards are a `main-api` domain entity (`gift_cards`),
  scoped by `storeId` like everything else. No new service needed.
- **Codes:** server-generated, 12-char uppercase, **unique per store** (index
  `{ storeId, code }`), not globally unique.
- **Redemption is atomic:** a single conditional `findOneAndUpdate` with
  `$inc`, so concurrent checkouts cannot overspend a card.
- **Discount cap:** `min(balanceCents, subtotalCents)`; leftover balance stays
  on the card for a future order.
- **Not snapshotted:** balances change at checkout, so they must never go into
  the static storefront snapshot — redemption validates live via
  `POST /public/checkout`. (This is the key cross-cutting constraint.)
- **Roles:** only `owner`/`manager` can issue or deactivate cards.

## 3. Per-service impact

### main-api
- New `gift-cards` module: `GiftCard` schema, service, controller.
- Endpoints: `GET /gift-cards`, `POST /gift-cards`, `PATCH /gift-cards/:id/deactivate`.
- Extend `CheckoutDto` with optional `giftCardCode`; apply redemption in the
  checkout flow; add `giftCardCode`/`discountCents` to `Order`.

### admin-frontend
- New `giftCards` Redux slice + `giftCardsApi`.
- New `/gift-cards` view: list (code, initial, balance, status), issue-by-amount
  modal, per-row deactivate.
- `/orders` view shows the applied discount per order.

### storefront-frontend
- Optional gift-card code field on the checkout form, sent as `giftCardCode`.
- Show discount line + remaining amount due from the returned `Order`.
- No client-side balance check (balances are not in the snapshot).

## 4. Open questions (logged in overview)

- Should the storefront expose a "check balance" endpoint before checkout? (#1)
- Per-store currency, or single-currency v1? Assume single-currency for now. (#2)

## 5. Out of scope (v1)

- Partial refunds back onto a gift card.
- Emailing the card code to a recipient.
- CSV export of gift cards.
