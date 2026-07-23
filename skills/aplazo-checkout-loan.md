---
name: Offer Aplazo Buy-Now-Pay-Later at checkout
description: Authenticate a merchant, originate an Aplazo installment loan for a cart, redirect the shopper to complete financing, then confirm loan status.
api: Aplazo Merchant Payment API (https://api.aplazo.mx)
source: https://github.com/aplazo/php.aplazo-magento-2-payment-gateway
operations:
  - POST /api/auth
  - POST /api/loan
  - GET /api/pos/loan/{cartId}
---

# Offer Aplazo BNPL at checkout

Use this to add Aplazo installment financing (Mexico, MXN only) as a
checkout option. All calls go to `https://api.aplazo.mx` in production or
`https://api.aplazo.net` in sandbox.

## Prerequisites
- `apiToken` and `merchantId` from the Aplazo merchant account.
- Every request also carries `merchant_id` and `api_token` headers.

## Steps
1. **Authenticate.** `POST /api/auth` with body
   `{ "apiToken": "...", "merchantId": "..." }`. Read the `Authorization`
   (JWT) from the response. It is short-lived — fetch a fresh one per session.
2. **Create the loan.** `POST /api/loan` with the `Authorization: <jwt>`
   bearer header and the cart payload (amount in MXN, items, return URLs).
   The response carries the redirect the shopper follows to complete
   financing on Aplazo.
3. **Send the shopper to Aplazo**, then wait for the callback/webhook (see
   `asyncapi/aplazo-webhooks.yml`) or poll.
4. **Confirm status.** `GET /api/pos/loan/{cartId}`. A status of
   `OUTSTANDING` means the loan is active — release the order.

## Rules
- Currency is **MXN** only.
- Do not fulfill until the loan reports `OUTSTANDING`.
- Refunds/cancellations are separate flows (`POST /api/pos/loan/refund`,
  `POST /api/pos/loan/cancel`); send `X-Idempotency-Key` on refunds so a retry
  never double-refunds. See `conventions/aplazo-conventions.yml`.
