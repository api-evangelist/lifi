---
name: Compare multiple LI.FI routes and execute a chosen one step by step
description: >-
  Use the advanced routing endpoints to fetch several competing routes, compare
  them on cost, speed and hop count, then execute the selected route one step at
  a time. For flows where the user should choose rather than accept the default.
api: openapi/lifi-openapi-original.yml
base_url: https://li.quest
operations:
  - GET /v1/tools
  - POST /v1/advanced/routes
  - POST /v1/advanced/stepTransaction
  - GET /v1/status
generated: '2026-07-19'
method: generated
source: >-
  openapi/lifi-openapi-original.yml (operations verified by path+method — the
  upstream spec defines no operationIds), https://docs.li.fi/agents/overview
---

# Compare routes and execute step by step

`GET /v1/quote` returns one route, already chosen for you. This skill uses the
advanced path instead, where several routes come back and the caller picks.
Reach for it when the tradeoff genuinely belongs to the user — a large transfer
where fees matter, a time-sensitive one, or where the user wants to avoid a
particular bridge.

## Quote vs Route

A **quote** is a single optimised route with transaction data already attached.
A **route** is a path that may span several steps, each of which needs its own
`POST /v1/advanced/stepTransaction` call before it can be signed. More control,
more round trips.

## Steps

### 1. Know the tool keys

`GET /v1/tools` returns `{bridges: [...], exchanges: [...]}`, each with a `key`
and a human-readable `name`. The `key` values are what the allow/deny/prefer
filters accept. Fetch this before filtering — do not guess key strings.

### 2. Request routes

`POST /v1/advanced/routes` with `fromChainId`, `toChainId`, `fromTokenAddress`,
`toTokenAddress`, `fromAmount` and `fromAddress`.

`options` accepts `order` (`RECOMMENDED`, `FASTEST`, `CHEAPEST`, `SAFEST`),
`slippage`, and the `allowBridges` / `preferBridges` / `denyBridges` /
`allowExchanges` / `preferExchanges` / `denyExchanges` lists.

The response carries `routes[]` and `unavailableRoutes`. Read
`unavailableRoutes.filteredOut` when the result set looks thin — it holds the
`ToolError` explaining why each tool declined, which is what you should tell the
user instead of "no routes found".

This endpoint shares the tight 75-per-two-hours budget with `/v1/quote`.

### 3. Compare on the numbers that differ

For each route compare `toAmount` and `toAmountMin` (what actually arrives),
`gasCostUSD`, the aggregate `feeCosts`, `steps.length` (each hop is a failure
opportunity), and `estimate.executionDuration`.

Present the real tradeoff. The cheapest route is often the slowest or the one
with the most hops; the fastest usually costs more. Do not silently pick — that
is what `/v1/quote` is for.

### 4. Execute step by step

For each step in the chosen route, in order:

1. `POST /v1/advanced/stepTransaction` with the step object to populate it with
   transaction data.
2. Check and set the ERC20 allowance for that step's `approvalAddress`.
3. Have the user sign and broadcast. **Confirm before each signature.**
4. Poll `GET /v1/status` with the resulting `txHash` until `DONE` before
   populating the next step.

Never populate or sign step N+1 before step N reaches `DONE`. The steps are
sequentially dependent, and the input amount of a later step is the realised
output of the earlier one.

### 5. Handle partial completion

A multi-step route can strand a user midway — the bridge lands but the
destination swap fails. The user then holds the intermediate token on the
destination chain. This is a `DONE` + `PARTIAL` or a `FAILED` after a successful
earlier step.

When it happens: report exactly which token the user is holding and on which
chain, then offer a fresh quote from that position. Do not retry the original
route from the beginning — the funds are no longer on the source chain.

## Notes

- Routes go stale like quotes. Re-request rather than executing a route object
  fetched long ago.
- `POST /v1/advanced/possibilities` is deprecated in the spec. Use
  `GET /v1/connections` and `GET /v1/tools` instead.
- Cross-chain execution is not atomic across hops; only individual same-chain
  swaps revert cleanly.
- Error envelope and code registry: `errors/lifi-error-codes.yml`.
- Rate limiting and the `ratelimit-*` response headers:
  `rate-limits/lifi-rate-limits.yml`.
