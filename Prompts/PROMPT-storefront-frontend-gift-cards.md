# PROMPT — storefront-frontend — Gift Cards

> **Target repo:** plaza-storefront-frontend
> **Branch:** feature/gift-cards
> **Prerequisites:** PROMPT-main-api-gift-cards.md (must be Verified)
> **Status:** Generated   <!-- Generated → Applied → Verified -->
> **Recommended model:** Haiku — one optional form field and a discount line; contracts fully specified

<!--
  Self-contained implementation prompt, generated from 01-storefront-frontend.md
  and Features/FEATURE-gift-cards.md (Step 3). Do not apply until the
  prerequisite main-api prompt is Verified — checkout must accept giftCardCode.
-->

---

## Context

You are working in the `plaza-storefront-frontend` repository (Next.js public
storefront). Buyers can now enter a gift-card code at checkout; the backend
applies the balance to the order. The storefront's only job is to send the code
and render the result — **all validation is server-side**, and gift-card
balances are never present in the static snapshot, so there is nothing to check
client-side.

**Pattern reference:** study the existing checkout page/form and the call it
makes to `POST /public/checkout` (store resolved via the `x-store-domain`
header). Do not touch snapshot consumption.

## Task

### 1. Checkout form

- Add an **optional** "Gift card code" text input to the checkout form.
- On submit, if the field is non-empty, include it in the existing `CheckoutDto`
  body as `giftCardCode` (string, sent as typed; the backend normalizes).
  If empty, omit the field entirely.

### 2. Order confirmation

- The returned `Order` now includes `discountCents` and `totalCents`
  (`subtotalCents - discountCents`).
- When `discountCents > 0`, render a discount line ("Gift card −{amount}") and
  the remaining amount due (`totalCents`).
- When a code was entered but `discountCents` is `0`, the order still succeeds
  — show a neutral notice that the code was not applied (invalid, inactive, or
  empty balance). Do not block or fail the checkout.

## Constraints

- No client-side balance check, code-format validation beyond trimming, or new
  API calls — redemption is validated live by `POST /public/checkout` only.
- Do not modify snapshot fetching or add gift-card data to any static path.

## Acceptance

- Checkout without a code behaves exactly as before (field omitted from body).
- A valid code produces a visible discount line and reduced total.
- An invalid/inactive code completes the order with no discount and shows the
  neutral notice.
