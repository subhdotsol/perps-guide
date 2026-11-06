# 07 — Open-source code to read

Repository existence, licence and last-push dates were checked against the GitHub API on 2026-11-12. Check the licence yourself before reusing anything: several of these are source-available, not open source.

## Read these first

| Order | Repo | Why |
|---|---|---|
| 1 | https://github.com/aeyakovenko/percolator-match | Tiny. Shows the matcher interface in one sitting |
| 2 | https://github.com/velocity-exchange/velocity-v1 | The most complete maintained Solana perp program you can read: programs, SDKs, keepers, ~120 tests, design notes in `docs/` |
| 3 | https://github.com/blockworks-foundation/mango-v4 | Fully on-chain order-book perps with cross-margin. Unmaintained but clearly written |
| 4 | https://github.com/gmsol-labs/gmx-solana | A live oracle-pool design on Solana, recently audited |
| 5 | https://github.com/aeyakovenko/percolator-prog | Wrapper around a verified risk engine; read the `v16_attack_*` tests |
| 6 | https://github.com/aeyakovenko/percolator | The engine and its spec (file 05) |

## Solana

| Repo | Design | Licence | State |
|---|---|---|---|
| https://github.com/velocity-exchange/velocity-v1 | Drift v2 fork on Anchor 1.0: keeper order book, JIT auction, AMM | Apache-2.0 | Active (Nov 2026) |
| https://github.com/velocity-exchange/protocol-v2 | Original Drift v2 (`drift-labs/protocol-v2` redirects here) | Apache-2.0 | Archived Jul 2026 |
| https://github.com/blockworks-foundation/mango-v4 | On-chain order-book perps + lending | MIT, GPL-3.0 for instructions | Stale since Apr 2025 |
| https://github.com/openbook-dex/openbook-v2 | Spot order book only | MIT, GPL-3.0 for instructions | Active |
| https://github.com/Ellipsis-Labs/phoenix-v1 | Crankless spot order book (not the perp program) | MIT | Active |
| https://github.com/Ellipsis-Labs/rise-public | SDK and account views for Phoenix perps; no public source for the perp program was found | MIT | Active |
| https://github.com/gmsol-labs/gmx-solana | Oracle-priced pool | BUSL-1.1 (source-available) | Active |
| https://github.com/solana-labs/perpetuals | Oracle-pool reference implementation; ancestor of several pool venues | Not detected | Archived 2023 |
| https://github.com/julianfssen/jupiter-perps-anchor-idl-parsing | Jupiter Perps interface parsing only | None | Stale |
| https://github.com/AdrenaFoundation/adrena-abi | Interface only | None | Stale |
| https://github.com/bulletxyz/bullet-exchange-interface | Interfaces and SDKs | MIT | Active |
| https://github.com/pacifica-fi | SDKs only | — | — |

**Not public:** the programs for Jupiter Perps, Pacifica, Phoenix perps, Flash Trade and Zeta. For those you get interfaces and docs.

## Other chains

| Repo | Design | Licence | State |
|---|---|---|---|
| https://github.com/dydxprotocol/v4-chain | Cosmos app-chain, order book inside validators | AGPL-3.0 with additional terms | Active |
| https://github.com/gmx-io/gmx-synthetics | Oracle pools with keeper execution | BUSL-1.1 | Active |
| https://github.com/elliottech/lighter-prover | ZK circuits for Lighter's matching | BUSL-1.1 | Active |
| https://github.com/nadohq/nado-contracts | Nado (ex-Vertex) | None | Active |
| https://github.com/0xOstium/smart-contracts-public | RWA oracle pool | MIT | May 2026 |
| https://github.com/equilibria-xyz/perennial-v2 | Oracle-settled perps | Not detected | Stale |
| https://github.com/Synthetixio/synthetix-v3 | Debt-pool perps | MIT | Archived |
| https://github.com/perpetual-protocol/perp-curie-contract | vAMM on Uniswap v3 | GPL-3.0 | Stale |
| https://github.com/hyperliquid-dex/node | Binary distribution only; the exchange core is closed | Apache-2.0 | — |
| https://github.com/hyperliquid-dex/hyperliquid-python-sdk | API client | MIT | — |

## Percolator family

| Repo | Role |
|---|---|
| https://github.com/aeyakovenko/percolator | Risk engine library, spec v16.9.1 |
| https://github.com/aeyakovenko/percolator-prog | Solana program wrapper |
| https://github.com/aeyakovenko/percolator-match | Reference matcher |
| https://github.com/aeyakovenko/percolator-cli | Command-line client |
| https://github.com/aeyakovenko/percolator-meta | Governance and reward bootstrap |
| https://github.com/aeyakovenko/percolator-stress-test | Load testing |
| https://github.com/dcccrypto/percolator | Engine fork (file 05) |
| https://github.com/dcccrypto/percolator-launch | "Pump.fun for perps" product |
| https://github.com/dcccrypto/percolator-perp-liquidity | Long-tail liquidity design and model |
| https://github.com/dcccrypto/percolator-portfolio | USDC cross-margin wrapper |
| https://github.com/dcccrypto/percolator-stake | Insurance staking |
| https://github.com/dcccrypto/percolator-keeper | Crank, oracle and liquidation bots |
| https://github.com/QuantuLabs/colada-engine | Independent derived risk engine |

All Apache-2.0 where a licence is stated, and all labelled unaudited.

## What to trace in each codebase

Do the same four exercises in every repo; the comparison is where the learning is.

1. **Open a position.** Find the instruction, the margin check, and where the position is written.
2. **Accrue funding.** Find the global index and the per-account settle.
3. **Liquidate.** Find the health check, who may call it, how the fee is split, and what happens when equity is negative.
4. **Update the price.** Find every place an oracle value is read and what validation surrounds it.

Then answer: which accounts does each instruction write? That tells you what can run in parallel and where the venue will bottleneck.

## Tooling you will need

- **Anchor** — the standard Solana program framework. Velocity is on 1.0.
- **LiteSVM** — fast in-process Solana test runtime; used by Velocity and the Percolator repos.
- **Kani** — Rust model checker used by Percolator. https://github.com/model-checking/kani
- **proptest** — property-based testing for Rust; pairs well with a reference model of your engine.
