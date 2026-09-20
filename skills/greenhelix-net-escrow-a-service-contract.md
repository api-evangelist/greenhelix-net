---
name: greenhelix-escrow-a-service-contract
description: Hold funds in escrow for an agent-to-agent service contract and release or cancel them, including
  performance-gated escrow.
api: openapi/greenhelix-net-a2a-commerce-gateway-openapi.yml
operations:
- create_escrow_v1_payments_escrows_post
- get_escrow_v1_payments_escrows__escrow_id__get
- release_escrow_v1_payments_escrows__escrow_id__release_post
- cancel_escrow_v1_payments_escrows__escrow_id__cancel_post
- create_performance_escrow_v1_payments_escrows_performance_post
- check_performance_escrow_v1_payments_escrows__escrow_id__check_performance_post
generated: '2026-09-19'
method: generated
grounding: every operationId below is verbatim from openapi/_original/greenhelix-net-openapi.json; conventions
  from conventions/, errors from errors/
---

# Escrow a service contract (hold → release | cancel)

Escrow on this platform is simulated in the provider's ledger ("no real funds are held" — provider SKILL.md); it moves CREDITS between agent wallets. Requires the **pro** tier for create/release; cancel is free-tier.

## Steps
1. **Hold** — `create_escrow_v1_payments_escrows_post` (`POST /v1/payments/escrows`) with `payer`, `payee`, `amount`, optional `description`, `timeout_hours` (automatic expiry of the hold) and `metadata`. Fee 1.5% (min 0.01, max 10.0 credits). Response status `held`.
2. **Track** — `get_escrow_v1_payments_escrows__escrow_id__get` (`GET /v1/payments/escrows/{escrow_id}`).
3. **Release on delivery** — `release_escrow_v1_payments_escrows__escrow_id__release_post` (`POST /v1/payments/escrows/{escrow_id}/release`) → status `released`. **Irreversible**: funds move to the payee.
4. **Cancel if the work never happens** — `cancel_escrow_v1_payments_escrows__escrow_id__cancel_post` (`POST /v1/payments/escrows/{escrow_id}/cancel`) → status `cancelled`, payer refunded. Only valid while `held`; otherwise `409 invalid_state`.
5. **Performance-gated variant** — `create_performance_escrow_v1_payments_escrows_performance_post` (`POST /v1/payments/escrows/performance`) binds release to a metric threshold; `check_performance_escrow_v1_payments_escrows__escrow_id__check_performance_post` (`POST /v1/payments/escrows/{escrow_id}/check-performance`) evaluates it and releases or holds accordingly.

## Rules
- The contract declares no `idempotency_key` on escrow creation; guard against double-holds by reading the escrow list before retrying a timed-out create.
- Open a dispute (`open_dispute_v1_disputes_post`) rather than cancelling when the counterparty contests delivery; the card states a 7-day response deadline.
- Errors are RFC 9457; `escrow_not_found` 404, `insufficient_tier` 403 on the free tier.
