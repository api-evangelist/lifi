---
name: Execute a cross-chain token transfer with LI.FI
description: >-
  Quote, execute and track a token transfer between two chains using the LI.FI
  routing API. Covers allowance checking, the unsigned-transaction handoff, and
  status polling to a terminal outcome.
api: openapi/lifi-openapi-original.yml
base_url: https://li.quest
operations:
  - GET /v1/chains
  - GET /v1/token
  - GET /v1/connections
  - GET /v1/quote
  - GET /v1/status
generated: '2026-07-19'
method: generated
source: >-
  openapi/lifi-openapi-original.yml (operations verified by path+method — the
  upstream spec defines no operationIds), https://docs.li.fi/agents/overview
---

# Execute a cross-chain token transfer

LI.FI aggregates bridges, DEXs and intent solvers behind one routing API. This
skill takes a transfer from intent to confirmed delivery.

**LI.FI never moves funds itself.** Every endpoint here is read-only. The API
returns an unsigned `transactionRequest`; you sign and broadcast it with the
user's wallet. Never claim a transfer succeeded on the strength of a quote.

## Authentication

An API key is optional but strongly recommended — anonymous callers get a much
lower rate limit. Send it as `x-lifi-api-key: <key>`. Keys come from
<https://li.fi/plans/>. See `authentication/lifi-authentication.yml`.

## Steps

### 1. Resolve chains and tokens

Call `GET /v1/chains` to confirm both chains are supported. Then resolve each
token with `GET /v1/token` (accepts a symbol or a contract address plus a
chain). Prefer resolving symbols dynamically over hardcoding addresses — the
same symbol has a different address on every chain, and `USDC` vs `USDC.e`
mistakes are the most common source of failed transfers.

The native token on EVM chains is `0x0000000000000000000000000000000000000000`.

### 2. Confirm a route exists (optional but cheap)

`GET /v1/connections` reports which token pairs are routable between two
chains. Checking here avoids burning `/v1/quote` budget, which has a tighter
rate limit (75 requests per two hours) than the other endpoints.

### 3. Request a quote

`GET /v1/quote` with `fromChain`, `toChain`, `fromToken`, `toToken`,
`fromAmount` and `fromAddress`. `fromAmount` is in the token's smallest unit —
6 decimals for USDC, 18 for ETH. Getting decimals wrong by a factor of 10^12 is
the second most common failure; read `decimals` from step 1 rather than
assuming.

Optional controls: `toAddress` (defaults to `fromAddress`), `slippage`
(default 0.005), `order` (`RECOMMENDED`, `FASTEST`, `CHEAPEST`, `SAFEST`), and
the `allowBridges` / `denyBridges` / `allowExchanges` / `denyExchanges` filters
keyed on values from `GET /v1/tools`.

The response carries `estimate` (expected output, `toAmountMin`, fee and gas
costs, duration) and `transactionRequest` (`to`, `data`, `value`, `gasLimit`,
`chainId`).

### 4. Check and set allowance

For ERC20 sources, the wallet must have approved `estimate.approvalAddress` for
at least `fromAmount`. If the allowance is short, the user must approve before
the transfer transaction will succeed. Native-token transfers need no approval.

Allowance is an on-chain read — do it via RPC or the LI.FI MCP server's
`get-allowance` tool, not via the REST API.

### 5. Sign and broadcast

Hand `transactionRequest` to the wallet. Capture the resulting source-chain
transaction hash. **Confirm with the user before signing** — this step spends
real funds and is irreversible.

### 6. Poll to a terminal state

`GET /v1/status` with `txHash`, plus `bridge`, `fromChain` and `toChain` from
the quote to speed lookup. Poll every 10-30 seconds. Do not poll faster; the
rate limit is per key across all endpoints.

Terminal handling:

| Status | Substatus | Meaning |
|---|---|---|
| `PENDING` | — | Keep polling |
| `DONE` | `COMPLETED` | Success, exact tokens received |
| `DONE` | `PARTIAL` | Success, a different token was received at full value |
| `DONE` | `REFUNDED` | Failed, funds returned to the user |
| `FAILED` | — | Inspect the error; may need user action |
| `NOT_FOUND` | — | Not yet indexed; keep polling briefly before treating as an error |

Report `PARTIAL` and `REFUNDED` to the user explicitly. Both are `DONE` but
neither is what the user asked for.

## Error handling

Errors return `{message, errors[]}` where each entry is a `ToolError` with
`errorType`, `code`, `tool` and `message` — an explanation per bridge or DEX
that declined. Full registry in `errors/lifi-error-codes.yml`.

- `NO_POSSIBLE_ROUTE` — try a different pair, chain or amount.
- `INSUFFICIENT_LIQUIDITY` — reduce the amount or allow more tools.
- `AMOUNT_TOO_LOW` / `FEES_HIGHER_THAN_AMOUNT` — increase the amount.
- `CANNOT_GUARANTEE_MIN_AMOUNT` — raise `slippage`.
- `TOOL_TIMEOUT` / `RPC_ERROR` — retry; the aggregator usually routes around it.
- HTTP `429` (code `1005`) — exponential backoff; read `ratelimit-reset` for
  the wait in seconds.

## Things to get right

- Same-chain swaps are atomic. Cross-chain transfers are **not** — the bridge
  leg can succeed while the destination swap fails, leaving the user with the
  bridged asset on the destination chain. Say so when it happens.
- `toAmountMin` is enforced on-chain. If the market moves past it the
  transaction reverts rather than filling at a worse price.
- Quotes go stale. Re-quote rather than signing a quote fetched minutes ago.
- There is no test mode. LI.FI retired testnet support and advises testing on
  mainnet with small amounts on low-gas L2s — see `sandbox/lifi-sandbox.yml`.
- There is no idempotency key. Safety comes from the chain: a signed
  transaction is nonce-bound and can only be included once. Never retry by
  re-signing without checking status first.
