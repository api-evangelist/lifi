# LI.FI

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
