# 02 — Jargon glossary

Keep this open while reading the rest. Grouped by topic, not alphabetically, so related terms sit together.

## Contract and pricing

| Term | Meaning |
|---|---|
| Perp / perpetual swap | Futures contract with no expiry, anchored to spot by funding |
| Dated future | Future with an expiry; converges to spot at settlement |
| Index price | Spot price aggregated from external venues |
| Oracle price | Index price as delivered on-chain by an oracle network |
| Mark price | Exchange's fair price used for PnL and liquidation |
| Basis | Perp price minus spot price |
| Premium | Basis as a fraction of spot |
| Funding rate | Periodic payment between longs and shorts, sized by premium |
| Impact price | Average price to fill a fixed notional on the book; used in premium calculations |
| TWAP / EMA | Time-weighted average / exponential moving average; smoothing against manipulation |
| Linear / inverse / quanto | Stable-margined / coin-margined / third-asset-margined contract |
| Hyperp / pre-launch perp | Perp with no external spot oracle; mark derives from its own trading |
| Power perp, everlasting option | Perp-style contracts with non-linear payoffs (Paradigm research) |

## Margin and positions

| Term | Meaning |
|---|---|
| Notional | Size × price |
| Leverage | Notional ÷ collateral |
| IM / IMR | Initial margin (requirement) — needed to open |
| MM / MMR | Maintenance margin (requirement) — needed to stay open |
| Equity / account value | Collateral + unrealised PnL − amounts owed |
| Health / margin ratio | Equity relative to maintenance requirement |
| Isolated / cross / portfolio margin | Per-position bucket / shared bucket / net-risk margining |
| Collateral haircut / weight | Discount applied to non-stable collateral |
| uPnL / rPnL | Unrealised / realised profit and loss |
| OI | Open interest: total outstanding contract size |
| Skew | Long OI minus short OI (non-zero only when a pool is counterparty) |
| OI cap | Maximum OI a market will accept |
| Reduce-only | Order that may only shrink a position |
| Position NFT | A position wrapped as a transferable token |

## Liquidation and insolvency

| Term | Meaning |
|---|---|
| Liquidation price | Price at which equity equals maintenance margin |
| Bankruptcy price | Price at which equity equals zero |
| Liquidation fee / penalty | Charged to the liquidated account; pays liquidator and insurance |
| Partial liquidation | Closing only enough to restore health |
| Liquidator / keeper | Bot that executes liquidations for a reward |
| Backstop liquidity | Vault that takes over positions the market cannot absorb (e.g. HLP) |
| Bad debt / deficit / shortfall | Loss beyond a bankrupt account's collateral |
| Insurance fund | Pool funded by fees that absorbs bad debt |
| Socialised loss | Bad debt spread across winners or all users |
| ADL (auto-deleveraging) | Force-closing profitable positions to cancel bankrupt ones |
| ADL queue | Ranking that decides who gets ADL'd first, usually profit × leverage |
| Clawback | Taking back already-credited profit |
| Haircut | Paying a claim at less than face value |
| Liquidation cascade | Liquidations moving price, triggering more liquidations |
| Circuit breaker / price band | Rule that halts or limits trading on extreme moves |

## Matching and liquidity

| Term | Meaning |
|---|---|
| CLOB | Central limit order book |
| Maker / taker | Adds resting liquidity / removes it |
| Spread, depth, slippage | Bid–ask gap, size available near mid, fill price vs expected |
| vAMM | Virtual AMM: a pricing curve with no real assets in it |
| Peer-to-pool / oracle-priced pool | Traders trade against an LP pool at oracle price (GMX, Jupiter Perps) |
| JLP, GLP, HLP | LP tokens/vaults of Jupiter, GMX, Hyperliquid |
| Borrow fee | Hourly fee pool venues charge instead of funding |
| Price impact fee | Fee simulating slippage on oracle-priced venues |
| RFQ / intent | Trader requests a quote; solvers or market makers respond |
| JIT liquidity | Makers get a short window to fill a taker order before an AMM does |
| Prop AMM | A market maker's private on-chain pricing program |
| Matcher (Percolator) | LP-chosen program called by CPI that returns price and size for a fill |
| Toxic flow / adverse selection | Orders from better-informed traders that lose money for makers |
| LVR | Loss-versus-rebalancing: what an AMM LP loses to arbitrageurs |
| Speed bump / cancel priority | Rules that let makers cancel before takers can hit stale quotes |
| Wash trading | Trading with yourself to inflate volume, often to farm points |

## Oracles

| Term | Meaning |
|---|---|
| Push oracle | Publisher writes prices on-chain continuously |
| Pull oracle | User fetches a signed price and submits it with the transaction |
| Confidence interval | Oracle's own uncertainty band (Pyth) |
| Staleness | Age of the last update; engines reject old prices |
| Oracle latency arbitrage | Trading on a price the oracle has not caught up to |
| Oracle manipulation | Moving a thin spot market to move the oracle |
| DEX oracle / pool TWAP | Price read from an on-chain AMM pool; the only option for unlisted tokens |

## Solana-specific

| Term | Meaning |
|---|---|
| Program / account | Stateless code / the data it owns |
| PDA | Program-derived address: an account only a program can sign for |
| CPI | Cross-program invocation: one program calling another |
| CU (compute units) | Per-transaction execution budget, hard cap 1.4M |
| Write lock | Only one transaction per block slot can write an account at a time; hot accounts bottleneck |
| Slab | One large account holding a whole market's state |
| Zero-copy | Reading account bytes in place instead of deserialising |
| Crank | Permissionless instruction that advances state (funding, liquidation, settlement) |
| Keeper | Off-chain bot that calls cranks |
| Priority fee / Jito tip | Paying to land a transaction sooner |
| ALT | Address lookup table; lets a transaction reference more accounts |
| Slot | ~400 ms block interval |
| Geyser / gRPC stream | Validator plugin streaming account changes to indexers |
| DLOB | Decentralised limit order book: orders stored on-chain, matched by keepers (Drift) |
| ACE | Application-controlled execution: the app, not the validator, sets ordering rules |
| BAM | Jito's Block Assembly Marketplace: programmable block building |

## Percolator-specific

| Term | Meaning |
|---|---|
| `V`, `C_tot`, `I` | Vault balance, total deposited capital, insurance |
| Senior / junior claim | Capital is senior; profit is junior and paid only if backed |
| `h` (haircut ratio) | Fraction of matured profit the vault can actually pay |
| Warmup / reserve `R_i` | New profit is locked before it counts as withdrawable |
| A / K / F indices | Global per-side coefficients: position scale, mark-and-loss accumulator, funding accumulator |
| DrainOnly → ResetPending → Normal | Side lifecycle after heavy ADL scaling |
| Price/funding envelope | Hard cap on how far one crank may move price |
| Source domain | An `(asset, side)` pair; profit from it is only as good as that side's backing |
| Credit rate | `min(1, backing / claims)` for a source domain |
| Lien | Reservation of backing against a specific use of profit |
| Backing provider | LP that posts capital to back a domain's winners |
| B residual | Bankruptcy shortfall booked to one domain |
| Wrapper | The Solana program around the engine: auth, oracles, token custody |
| Kani | Rust model checker used to prove engine invariants |

## Market-structure slang

| Term | Meaning |
|---|---|
| Basis trade / cash-and-carry | Long spot, short perp, collect funding |
| Delta-neutral | Net zero price exposure |
| Funding arb | Opposite positions on two venues with different funding rates |
| Points farming | Trading to earn airdrop points rather than for PnL |
| Builder codes | Fee-sharing identifiers for front-ends routing to a venue |
| HIP-3 | Hyperliquid's permissionless perp-market deployment standard |
| Perp DEX aggregator | Router that splits an order across venues |
| ICM | "Internet capital markets": Solana's framing for on-chain trading of everything |
