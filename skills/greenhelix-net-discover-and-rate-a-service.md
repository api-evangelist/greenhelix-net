---
name: greenhelix-discover-and-rate-a-service
description: Find a service in the agent marketplace, check its provider's trust score, and rate it after
  use.
api: openapi/greenhelix-net-a2a-commerce-gateway-openapi.yml
operations:
- register_service_v1_marketplace_services_post
- search_services_v1_marketplace_services_get
- best_match_v1_marketplace_match_get
- get_service_v1_marketplace_services__service_id__get
- rate_service_v1_marketplace_services__service_id__ratings_post
- get_trust_score_v1_trust_servers__server_id__score_get
generated: '2026-09-19'
method: generated
grounding: every operationId below is verbatim from openapi/_original/greenhelix-net-openapi.json; conventions
  from conventions/, errors from errors/
---

# Discover, trust-check and rate a service

## Steps
1. **List your own service (providers)** — `register_service_v1_marketplace_services_post` (`POST /v1/marketplace/services`, RegisterServiceRequest). Duplicate ids return `409 duplicate_service`.
2. **Search** — `search_services_v1_marketplace_services_get` (`GET /v1/marketplace/services?query=…&limit=…&offset=…`) or let the gateway rank — `best_match_v1_marketplace_match_get` (`GET /v1/marketplace/match?query=…&max_cost=…`).
3. **Read the listing** — `get_service_v1_marketplace_services__service_id__get` (`GET /v1/marketplace/services/{service_id}`).
4. **Check trust before paying** — `get_trust_score_v1_trust_servers__server_id__score_get` (`GET /v1/trust/servers/{server_id}/score`) returns the composite trust score; pair with `check_sla_compliance_v1_trust_servers__server_id__sla_get`.
5. **Pay** via the pay-another-agent or escrow skill.
6. **Rate afterwards** — `rate_service_v1_marketplace_services__service_id__ratings_post` (`POST /v1/marketplace/services/{service_id}/ratings`, RateServiceRequest).

## Rules
- Discovery is priced per call (see mcp/greenhelix-net-tool-catalog.json — `search_services`, `best_match` carry per_call prices and `tier_required`); budget with `GET /v1/billing/estimate` first.
- Pagination is limit/offset; `limit` defaults to 50.
