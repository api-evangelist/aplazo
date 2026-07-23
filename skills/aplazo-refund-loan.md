---
name: Refund or cancel an Aplazo loan
description: Reverse an Aplazo installment loan — cancel an incomplete loan or issue an idempotent refund against an outstanding one.
api: Aplazo Merchant Payment API (https://api.aplazo.mx)
source: https://github.com/aplazo/php.aplazo-magento-2-payment-gateway
operations:
  - POST /api/pos/loan/refund
  - POST /api/pos/loan/cancel
  - GET /api/pos/loan/{cartId}
---

# Refund or cancel an Aplazo loan

## Prerequisites
- `apiToken` + `merchantId`; send `merchant_id` and `api_token` headers on
  every request. Host `https://api.aplazo.mx` (live) / `https://api.aplazo.net`
  (sandbox).

## Cancel (loan not completed/paid)
1. `POST /api/pos/loan/cancel` with the cart identifier.
2. Optionally confirm with `GET /api/pos/loan/{cartId}`.

## Refund (loan already OUTSTANDING)
1. Generate a stable idempotency key for this refund attempt.
2. `POST /api/pos/loan/refund` with the cart id + `totalAmount` and the header
   `X-Idempotency-Key: <key>`.
3. Retry with the **same** key on any network failure — Aplazo dedupes so the
   refund is applied at most once.

## Rules
- Never retry a refund with a new idempotency key.
- Amounts are MXN. See `conventions/aplazo-conventions.yml` and
  `errors/aplazo-problem-types.yml`.
