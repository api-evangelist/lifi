# LI.FI

LI.FI is the routing and execution layer for cross-chain liquidity, payments, swaps and yield. A single integration gives an application access to more than 100 aggregated bridges, DEXs, DEX aggregators and intent-based solvers across 58+ chains spanning EVM, Solana, Bitcoin, SUI and TRON.

The LI.FI API returns an optimal route plus a ready-to-sign, unsigned transaction — LI.FI never takes custody. The product set extends to Composer for one-click DeFi deposits across 20+ protocols, Earn for yield discovery and portfolio tracking, an intent/solver marketplace built on the Open Intents Framework, an embeddable Widget, a TypeScript SDK, a CLI and a hosted MCP server.

- Website — https://li.fi
- Documentation — https://docs.li.fi
- API reference — https://docs.li.fi/api-reference/introduction
- Status — https://status.li.fi
- GitHub — https://github.com/lifinance

## APIs

| API | Base URL | Spec |
|---|---|---|
| LI.FI API | `https://li.quest` | `openapi/lifi-openapi-original.yml` |
| LI.FI Earn Data API | `https://earn.li.fi` | `openapi/lifi-earn-openapi-original.yml` |
| LI.FI Intents Order Server API | `https://order.li.fi` | docs only |

## Artifacts

| Directory | What it holds |
|---|---|
| `openapi/` | LI.FI API (OpenAPI 3.0.2, 28 paths) and Earn API (3.0.0, 5 paths) specs |
| `overlays/` | API Evangelist enhancements over each spec, including the securityScheme LI.FI omits |
| `authentication/` | API key profile across API, Earn, MCP and Intents surfaces |
| `mcp/` | Hosted MCP server at `https://mcp.li.quest/mcp` and its 19 published tools |
| `skills/` | Three packaged Agent Skills — transfer, yield deposit, route comparison |
| `cli/` | `@lifi/cli` command surface |
| `packages/` | Five first-party npm packages, verified against the registry |
| `components/` | LI.FI Widget and the hosted surfaces (Playground, Explorer, Partner Portal) |
| `llms/` | LI.FI's published `llms.txt` |
| `well-known/` | `/.well-known/` probe results and the AI plugin descriptor |
| `errors/` | Documented error-code registry (1000–1013 plus tool errors) and derived problem types |
| `rate-limits/` | Per-endpoint limits, the two-hour rolling window model and `ratelimit-*` headers |
| `conventions/` | Cross-cutting request/response semantics |
| `lifecycle/` | Versioning, deprecations, removals, status page |
| `changelog/` | Structured recent entries from the monthly changelog |
| `sandbox/` | Environments and LI.FI's deliberate no-testnet testing policy |
| `data-model/` | Entity graph derived from both specs |
| `conformance/` | Standards conformance, asserted and refuted with evidence |
| `security/` | Domain security probe, bug bounty and disclosure program |
| `agentic-access/` | Recommended `x-agentic-access` contracts for 33 operations |

## Notes

- **No idempotency contract.** LI.FI documents no idempotency key. Safety is on-chain: a signed transaction is nonce-bound, and the enforced minimum-received amount reverts rather than filling badly.
- **No test mode.** LI.FI retired testnet support because bridges and DEXs lack testnet liquidity, and advises testing on mainnet with small amounts on low-gas L2s.
- **No OAuth.** Authentication is a single static `x-lifi-api-key` header, so no `scopes/` artifact exists.
- **No published certifications.** Assurance is independent smart contract audits, annual Web2 penetration testing and a $1M Cantina bug bounty — not SOC 2 or ISO 27001.
- **No event surface.** No AsyncAPI and no webhook catalog are published, so neither is claimed.
