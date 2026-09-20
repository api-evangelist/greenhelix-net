---
name: greenhelix-pay-another-agent
description: Pay another agent with an authorize-then-capture payment intent on the A2A Commerce Gateway,
  with an idempotency key and a refund path.
api: openapi/greenhelix-net-a2a-commerce-gateway-openapi.yml
operations:
- register_v1_register_post
- get_balance_v1_billing_wallets__agent_id__balance_get
- create_intent_v1_payments_intents_post
- get_intent_v1_payments_intents__intent_id__get
- capture_intent_v1_payments_intents__intent_id__capture_post
- refund_intent_v1_payments_intents__intent_id__refund_post
generated: '2026-09-19'
method: generated
grounding: every operationId below is verbatim from openapi/_original/greenhelix-net-openapi.json; conventions
  from conventions/, errors from errors/
---

# Pay another agent (intent → capture → refund)

Base URL `https://api.greenhelix.net` (sandbox: `https://sandbox.greenhelix.net`, resets on deploy).
Auth: `Authorization: Bearer a2a_{tier}_{24hex}` (or `X-API-Key`). Every error is RFC 9457 `application/problem+json`.

## Steps
1. **Get a key and wallet** — `register_v1_register_post` (`POST /v1/register`, body `{"agent_id": "<your-agent>"}`, no auth). Returns a free-tier key and a wallet with 500 credits. Keep the key; it is shown once.
2. **Check funds** — `get_balance_v1_billing_wallets__agent_id__balance_get` (`GET /v1/billing/wallets/{agent_id}/balance?currency=CREDITS`). A 402 `insufficient_balance` on the next step means deposit first.
3. **Authorize** — `create_intent_v1_payments_intents_post` (`POST /v1/payments/intents`) with `payer`, `payee`, `amount` (number or decimal string, ≤ 8 fractional digits), `currency` (default CREDITS) and **always** an `idempotency_key` derived from your own order id. Retrying with the same key returns the same intent; a different payload under the same key is `409 duplicate_intent`. A 2% gateway fee is charged here.
4. **Inspect** — `get_intent_v1_payments_intents__intent_id__get` (`GET /v1/payments/intents/{intent_id}`) — status must be `pending` before capture.
5. **Capture** — `capture_intent_v1_payments_intents__intent_id__capture_post` (`POST /v1/payments/intents/{intent_id}/capture`). The transition PENDING→CAPTURED is an atomic compare-and-set: a second capture returns `409 invalid_state`, so a capture retry is safe.
6. **Undo if needed** — `refund_intent_v1_payments_intents__intent_id__refund_post` (`POST /v1/payments/intents/{intent_id}/refund`): voids a pending intent or reverse-transfers a settled one. No time window is published. Reconcile against `fee_retained`/`fee_policy` in the response — ADR-011 says the 2% fee is retained; the older reference page says it is credited back.

## Rules
- Honour `X-RateLimit-Remaining`/`X-RateLimit-Reset`; on 429 back off for `X-RateLimit-Reset` seconds. 400/401/402/403/404/409 are not retryable.
- Log `X-Request-ID` from every response for support.
- Never call `POST /v1/execute` — it is 410 Gone (sunset 2026-10-01); use the routes above.
