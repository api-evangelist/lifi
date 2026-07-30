---
name: Discover a yield vault and deposit into it with LI.FI Earn and Composer
description: >-
  Find the best DeFi yield vault for an asset using the LI.FI Earn Data API,
  then deposit into it from any chain in one transaction via Composer, and track
  the resulting position.
api: openapi/lifi-earn-openapi-original.yml
secondary_api: openapi/lifi-openapi-original.yml
base_url: https://earn.li.fi
operations:
  - ChainsController_getChains_v1
  - ProtocolsController_getProtocols_v1
  - VaultsController_listVaults_v1
  - VaultsController_getVault_v1
  - PortfolioController_getPositions_v1
  - GET /v1/quote
  - GET /v1/status
generated: '2026-07-19'
method: generated
source: >-
  openapi/lifi-earn-openapi-original.yml (operationIds verified verbatim),
  openapi/lifi-openapi-original.yml, https://docs.li.fi/earn/recipes/discover-and-deposit
---

# Discover a yield vault and deposit into it

Two APIs cooperate here. **Earn** (`https://earn.li.fi`) is a read-only
discovery layer over indexed DeFi vaults. **Composer**, reached through the
normal routing API at `https://li.quest`, does the execution. They join on one
value: the vault's contract address.

## Authentication

The Earn API declares `x-lifi-api-key` as required in its spec. Send it as a
header on every Earn call. The routing API accepts the same key.

## Steps

### 1. Scope the search

`ChainsController_getChains_v1` (`GET /v1/chains`) lists chains with at least
one indexed vault. `ProtocolsController_getProtocols_v1` (`GET /v1/protocols`)
lists the indexed protocols — Aave, Morpho, Euler, Pendle, Compound and others.

Only query chains and protocols that appear here. A chain LI.FI can route to is
not necessarily a chain Earn has indexed.

### 2. List candidate vaults

`VaultsController_listVaults_v1` (`GET /v1/vaults`).

Filters: `chainId`, `asset`, `protocol`, `isTransactional`, `isRedeemable`,
`isComposerSupported`. Sorting: `apy`, `tvl`.

**Set `isComposerSupported=true` and `isTransactional=true` whenever the goal is
to deposit.** Without them you will surface vaults you cannot actually deposit
into, and the failure only shows up several steps later at quote time.

Add `isRedeemable=true` if the user will want to withdraw through LI.FI. Some
protocols are deposit-only and must be exited through their own interface.

### 3. Judge the vault, not just the APY

Each vault returns a uniform shape (named `Vault` in the spec,
`NormalizedVault` in the docs): `address`, `chainId`, `network`, `slug`,
`protocol`, `underlyingTokens`, plus `analytics.apy.{base,reward,total}`,
`apy1d`, `apy7d`, `apy30d` and `tvl.usd`.

Do not rank on `apy.total` alone. Compare it against `apy7d` and `apy30d` — a
headline rate far above the trailing averages is usually a transient incentive.
Check `tvl.usd` for depth, and separate `apy.base` from `apy.reward` so the user
knows how much of the yield depends on token emissions continuing.

Use `VaultsController_getVault_v1` (`GET /v1/vaults/{chainId}/{address}`) for
the full detail on a shortlisted vault.

### 4. Present the choice

Show the user the vault, protocol, chain, base vs reward APY split, trailing
averages and TVL, and get an explicit decision before quoting. This step commits
real capital to a third-party protocol; LI.FI is the router, not the
counterparty, and the user is taking that protocol's risk.

### 5. Quote the deposit

Call `GET /v1/quote` on `https://li.quest` with the **vault address as
`toToken`**. Composer activates automatically — there is no separate endpoint
and no separate integration.

- Same-chain deposit: `fromChain` and `toChain` both the vault's chain.
- Cross-chain deposit: any supported source chain and token; LI.FI bridges,
  swaps and deposits in one flow.

The response contains a normal `transactionRequest`.

### 6. Approve, sign, broadcast

Same as any transfer: check the ERC20 allowance against
`estimate.approvalAddress`, approve if short, then have the user sign. Confirm
before signing — this spends real funds.

### 7. Track and verify the position

Poll `GET /v1/status` until `DONE` or `FAILED`, exactly as in the transfer
skill. Then confirm the position landed with
`PortfolioController_getPositions_v1`
(`GET /v1/portfolio/{userAddress}/positions`), which returns the user's vault
positions across every indexed protocol.

## Constraints that will bite you

- **EVM chains only.** Composer does not support Solana or other non-EVM chains,
  even though the routing API does.
- **Tokenised positions only.** The protocol must return a token on deposit. A
  vault that does not mint a receipt token cannot be composed.
- **Deposit-only protocols exist.** Check `isRedeemable` before promising the
  user they can withdraw the same way.
- **The legacy Composer stack is gone.** `POST /route` and `GET /zap-packs` were
  removed in June 2026. Use the current compose endpoints — see
  `changelog/lifi-changelog.yml`.
- **Quote budget is tight.** `/v1/quote` allows 75 requests per two hours. Do
  vault filtering on the Earn API, which is not on that budget, and quote only
  the vault the user actually chose.

## Errors

Earn returns a `ValidationError` on a bad request and `NotFoundError` when no
indexed vault exists at that chain and address. Routing errors follow the
standard `{message, errors[]}` envelope — see `errors/lifi-error-codes.yml`.
