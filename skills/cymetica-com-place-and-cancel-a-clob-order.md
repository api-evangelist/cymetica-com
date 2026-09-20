---
generated: '2026-09-19'
method: generated
name: Place and cancel a CLOB order
description: Register an agent key, discover a tradable pair and its book, place a limit order with replay protection, then amend or cancel it and read the resulting position — grounded in the EventTrader Public API.
api: openapi/cymetica-com-eventtrader-public-api-openapi.yml
operations: [register_new_agent_mcp_v1_register_post, list_exchange_pairs_api_v1_exchange_pairs_get, orderbook_api_v1_exchange__symbol__book_get, best_bid_offer_api_v1_exchange__symbol__bbo_get, place_order_api_v1_exchange__symbol__orders_post, cancel_all_orders_for_pair_api_v1_exchange__symbol__orders_delete, place_order_api_v1_clob_orders_post, amend_order_api_v1_clob_orders__order_id__patch, cancel_order_api_v1_clob_orders__order_id__delete, cancel_all_orders_api_v1_clob_orders_delete, get_portfolio_positions_api_v1_portfolio_positions_get]
source: >-
  Grounded in openapi/cymetica-com-eventtrader-public-api-openapi.yml, the provider's curated public
  OpenAPI captured 2026-09-19 at https://cymetica.com/openapi-public.json (source in openapi/_original/).
  Every operationId verified verbatim in that spec. Auth per authentication/cymetica-com-authentication.yml,
  errors per errors/cymetica-com-problem-types.yml, idempotency + reversibility per
  conventions/cymetica-com-conventions.yml, limits per rate-limits/cymetica-com-rate-limits.yml.
---

# Place and cancel a CLOB order

EventTrader exposes two order surfaces in its public spec: the per-pair exchange router
(`/api/v1/exchange/{symbol}/orders`) and the account-wide CLOB router (`/api/v1/clob/orders`).
Only the exchange router documents an `Idempotency-Key`; the CLOB router is the one with amend
and per-order cancel. Read `conventions/cymetica-com-conventions.yml` before mixing them.

## Before you trade

- **Paper first.** Every order request accepts `"mode": "paper"` (documented on /api-docs, "Paper
  Trading"); paper orders return `sim-{uuid}` transaction hashes and draw on a simulated balance.
  Nothing in this skill moves real funds unless you drop the paper flag.
- **You pay your own gas.** The provider's stated policy: user-initiated transactions are never
  platform-funded. Fund the agent account with its own USDC before going live.
- **Base URL** is `https://cymetica.com` (spec `servers[0]`); there is no separate api host.

## Auth
- Self-serve agent key: `register_new_agent_mcp_v1_register_post` (`POST /mcp/v1/register`, no auth)
  returns an `mcp_`-prefixed key. Send it as `X-API-Key` on every call.
- Alternatively a user session JWT (`Authorization: Bearer`) or an OAuth 2.0 bearer with the `trade`
  scope; see `authentication/cymetica-com-authentication.yml` and `scopes/cymetica-com-scopes.yml`.

## Steps

1. **Find the pair** — `list_exchange_pairs_api_v1_exchange_pairs_get` (`GET /api/v1/exchange/pairs`).
   Use the returned `symbol` verbatim; the spec's servers and the MCP `list_clob_pairs` tool agree.
2. **Read the book** — `orderbook_api_v1_exchange__symbol__book_get` (`GET /api/v1/exchange/{symbol}/book`)
   and `best_bid_offer_api_v1_exchange__symbol__bbo_get` for the touch. Price limit orders inside the spread
   to earn the maker rebate (fee schedule in `plans/cymetica-com-plans-pricing.yml`).
3. **Place with replay protection** — `place_order_api_v1_exchange__symbol__orders_post`
   (`POST /api/v1/exchange/{symbol}/orders`). Send an `Idempotency-Key` header: per the spec the response
   is cached for 300 s keyed by `(user_id, idempotency_key)` and a retry with the same key returns the
   cached response without re-executing. Generate the key BEFORE the first attempt and reuse it on
   timeout. This is the only write in the public spec with a documented idempotency mechanism.
4. **Or place on the CLOB router** — `place_order_api_v1_clob_orders_post` (`POST /api/v1/clob/orders`,
   body `src__api__routes__clob__PlaceOrderRequest`). No idempotency header is documented here: on a
   timeout, list open orders before retrying rather than re-sending.
5. **Amend** — `amend_order_api_v1_clob_orders__order_id__patch` (`PATCH /api/v1/clob/orders/{order_id}`,
   body `AmendOrderRequest`).
6. **Cancel** — `cancel_order_api_v1_clob_orders__order_id__delete` (`DELETE /api/v1/clob/orders/{order_id}`)
   for one order; `cancel_all_orders_api_v1_clob_orders_delete` (`DELETE /api/v1/clob/orders`) for all;
   `cancel_all_orders_for_pair_api_v1_exchange__symbol__orders_delete` for one pair on the exchange router.
   A cancel is the reversal path for an unfilled order. The provider states NO window for it and a
   filled order cannot be un-filled — close the position with an opposite-side order instead.
7. **Verify** — `get_portfolio_positions_api_v1_portfolio_positions_get` (`GET /api/v1/portfolio/positions`).

## Errors and limits
- `401 {"detail": "Authentication required. Provide Bearer token or X-API-Key header."}` with
  `WWW-Authenticate: Bearer` when the key is missing (observed live).
- `422 HTTPValidationError` (`detail[]` of `{loc, msg, type}`) on a malformed body — every write in the
  spec declares it.
- `429 Too Many Requests` on limit exhaustion; read `X-RateLimit-Limit`, `X-RateLimit-Remaining`,
  `X-RateLimit-Reset` (Unix timestamp). Order placement has its own per-tier budget (3,000 orders/min on
  a standard key) separate from the 100 req/min general limit.
