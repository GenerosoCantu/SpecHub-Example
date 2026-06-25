# PROMPT — main-api — Gift Cards

> Self-contained implementation prompt for the **`main-api`** repository.
> Generated from `03-main-api.md` and `Features/FEATURE-gift-cards.md`.
> AI-generated, human-reviewed, disposable — archived to `Prompts/Implemented/`
> after the work lands. Run it in a fresh agent session inside the `main-api`
> repo. It is deliberately a tight slice — it does not ask you to read the whole
> repo, only the one pattern reference named below.

---

## Context

You are working in the `main-api` repository (NestJS 10, TypeScript, MongoDB via
Mongoose, Atlas database `plaza`). Every domain entity is scoped by the `storeId`
claim on the JWT. Follow the existing module conventions exactly.

**Pattern reference:** study `src/products/` — it is the canonical example of a
feature module here (`*.module.ts`, `*.service.ts`, `*.controller.ts`,
`*.schema.ts`, `dto/`). Mirror its structure, guard usage, and response shapes.

## Task

Add a **Gift Cards** feature.

### 1. Schema — `src/gift-cards/gift-card.schema.ts`

Collection `gift_cards`:

| Field          | Type    | Required | Default  | Notes                         |
| -------------- | ------- | -------- | -------- | ----------------------------- |
| `storeId`      | string  | yes      | —        | indexed                       |
| `code`         | string  | yes      | —        | 12-char uppercase             |
| `initialCents` | number  | yes      | —        |                               |
| `balanceCents` | number  | yes      | =initial |                               |
| `active`       | boolean | yes      | `true`   |                               |
| `createdAt`    | Date    | yes      | now      |                               |

Add a **compound unique index** on `{ storeId, code }` (per-store unique, not
global).

### 2. Endpoints — `src/gift-cards/gift-cards.controller.ts`

| Method | Path                         | Guard                              | Body / Result                         |
| ------ | ---------------------------- | ---------------------------------- | ------------------------------------- |
| GET    | `/gift-cards`                | `JwtAuthGuard`                     | `{ data: GiftCard[] }` scoped by store |
| POST   | `/gift-cards`                | `JwtAuthGuard` + `RolesGuard('owner','manager')` | `IssueGiftCardDto { initialCents }` → `GiftCard` |
| PATCH  | `/gift-cards/:id/deactivate` | `JwtAuthGuard` + `RolesGuard('owner','manager')` | → updated `GiftCard` (`active:false`) |

- On issue, generate `code` server-side (12-char uppercase, retry on the rare
  unique-index collision) and set `balanceCents = initialCents`.

### 3. Checkout redemption — extend `src/orders/`

- Add optional `giftCardCode?: string` to `CheckoutDto`.
- In the checkout flow, when a code is present, redeem **atomically**:

  ```ts
  const amount = Math.min(card.balanceCents, subtotalCents);
  const updated = await this.giftCardModel.findOneAndUpdate(
    { code, storeId, active: true, balanceCents: { $gte: amount } },
    { $inc: { balanceCents: -amount } },
    { new: true },
  );
  ```

  If no document matches (missing / inactive / insufficient), apply **no**
  discount and do not fail the order — `discountCents = 0`.
- Persist `giftCardCode` and `discountCents` on the `Order`; set
  `totalCents = subtotalCents - discountCents`.

## Constraints

- Scope every gift-card query by `storeId` (use the `@CurrentStore()` decorator).
- Do **not** add gift-card balances to the published snapshot — redemption is
  always live.
- Do not modify auth, tenant resolution, or the asset service.

## Acceptance

- Issuing returns a unique 12-char code with `balanceCents == initialCents`.
- Two concurrent checkouts against the same card cannot drive the balance
  negative.
- Discount never exceeds the order subtotal; leftover balance remains on the card.
- Deactivated cards cannot be redeemed.
