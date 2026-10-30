# 06 — Established venues

Snapshot of 2026-11-12. Open interest (OI) is from DefiLlama's open-interest API and 24-hour volume from CoinGecko's derivatives API, both single-day readings. The two sources sometimes disagree, and figures marked "reported" come from press coverage rather than a primary source.

- OI data: https://api.llama.fi/overview/open-interest
- Volume data: https://api.coingecko.com/api/v3/derivatives/exchanges

Total perp OI tracked by DefiLlama on the day: about $16.35B.

## What changed in 2026

If you learned this space before mid-2026, these are the facts most likely to be out of date:

- **Drift is now Velocity**, rebuilt after a ~$285M governance compromise.
- **Flash Trade wound down.** Adrena is in maintenance mode. Mango closed in 2025.
- **GMTrade (a GMX port) is the largest Solana venue by OI.**
- **Jupiter added an order-book engine** beside its pool.
- **Switchboard shut down** and **Pyth's standard feeds became a paid service** (file 08).
- **Hyperliquid's builder-deployed markets** now hold more OI than any venue except Hyperliquid itself.

## Solana

| Venue | Design | Status | OI | Notes |
|---|---|---|---|---|
| **GMTrade** | Oracle-priced pool (GMX v2 port) | Alive | $168M | Largest on Solana by OI. Commodity perps via Chainlink Data Streams. Source-available |
| **Pacifica** | Order book, off-chain matching (reported), Solana settlement | Alive | $53M | Largest on Solana by volume (~$781M/day). Closed source; architecture not documented in a primary page |
| **Jupiter Perps** | JLP pool for majors + "GUM" order book for six beta markets | Alive | $45M | Oracle is Chaos Labs' Edge with Chainlink and Pyth as checks. No permissionless listing |
| **Phoenix perps** | On-chain order book + "spline" quote curves | Alive, growing | $16M | Mark is a median of oracle, book and external price. Stock perps since June 2026 |
| **BULK** | Order book on its own validator-run network, settling on Solana | Early | $3–14M (sources differ) | 25 ms taker speed bump, portfolio margin |
| **Bullet** (ex-Zeta) | Order book on a Solana "network extension" rollup | Alive | $4M | Sequencer-based, built on Sovereign SDK |
| **Velocity** (ex-Drift) | Drift v2 fork: keeper order book + AMM | Rebuilt, tiny | $0.2M | New program, no state carried over, USDT-settled. Order book and prop AMM listed as not yet shipped |
| **Flash Trade** | Oracle pool, later on an ephemeral rollup | Wound down Aug 2026 | $0 | Shut five weeks after its V2 launch |
| **Adrena** | Oracle pool | Maintenance mode | — | Development halted |
| **Mango v4** | On-chain order book + lending | Closed Jan 2025 | — | Code remains a good read |
| **Ranger** | Aggregator | Conflicting reports | — | One source reports a wind-down in May 2026; a project page still lists it as active. Verify before relying on it |
| **Arqen** | Isolated community-funded pools for memecoins | Live Sep 2026, toy-sized | — | See file 10 |

Sources:
- Velocity: https://github.com/velocity-exchange/velocity-v1/blob/master/docs/DRIFT-TO-VELOCITY.md · https://solanacompass.com/news/drift-co-founder-cindy-leow-steps-down-hands-rebuilt-velocity-exchange-to-new-team
- Jupiter: https://docs.jup.ag/user-docs/trade/perps/technical-reference · https://solanacompass.com/news/jupiter-perps-adds-six-markets-including-tokenized-spacex-hype-and-zec-via-gum-orderbook
- GMTrade: https://solanacompass.com/news/gmtrade-opens-gold-silver-and-wti-crude-oil-perpetuals-on-solana-with-247-trading
- Pacifica: https://docs.pacifica.fi/pacifica/readme
- Phoenix: https://docs.phoenix.trade/phoenix/matching-engine/matching-engine.md · https://docs.phoenix.trade/phoenix/margin-and-risk/mark-price.md · https://docs.phoenix.trade/phoenix/margin-and-risk/liquidations.md
- BULK: https://docs.bulk.trade/llms.txt
- Bullet: https://docs.bullet.xyz/ · https://blockworks.co/news/solana-l2-cex-perps-competition
- Flash: https://solanacompass.com/news/flash-trade-winds-down-solana-perpetuals-exchange-after-three-years-seeks-acquirer
- Adrena: https://www.kucoin.com/news/flash/solana-perpetual-dex-adrena-enters-maintenance-mode
- Mango: https://www.theblock.co/post/334172/mango-markets-to-wind-down-in-wake-of-sec-settlement-dao-battle
- Ranger: https://solanafloor.com/news/ranger-finance-winds-down · https://solanacompass.com/projects/ranger-finance

### What the Solana table tells you

- **The survivors with volume moved matching off the base chain** (Pacifica, BULK, Bullet) or use oracle pricing (GMTrade, Jupiter). Phoenix is the main fully on-chain order book.
- **Pool venues died of economics, not exploits.** Flash and Adrena closed without being hacked.
- **The biggest loss was governance, not code** (Drift).
- **Nobody here lists permissionlessly.** That is the gap file 10 is about.

### Two designs worth studying closely

**Phoenix "splines".** A market maker posts one quote curve instead of a ladder of orders. Updating it costs about 600 compute units, against roughly 300,000 for the equivalent orders. This is a direct answer to the "makers cannot requote fast enough on-chain" problem from file 03.

**Jupiter's oracle stack.** One primary oracle with two others as verification and fallback. Worth copying as a pattern regardless of your design.

## Elsewhere

| Venue | Design | OI | Notes |
|---|---|---|---|
| **Hyperliquid** | Order book on its own chain; protocol vault (HLP) as backstop | $6.62B | Dominant. ~$11B/day. Node software is closed source |
| **trade.xyz** (on Hyperliquid) | Builder-deployed market (HIP-3), equities and RWA | $4.39B | About 97% of builder-deployed volume |
| **Aster** | Off-chain order book | $1.32B | ~$2B/day |
| **Variational** | RFQ | $807M | The largest RFQ-style venue |
| **Lighter** | ZK rollup on Ethereum, central sequencer, proven matching | $492M + $190M | Also runs Robinhood Chain's perp engine |
| **GRVT** | Hybrid | $380M | |
| **edgeX** | Own chain | $151–305M (sources differ) | |
| **Arcus** | dYdX Labs on Robinhood Chain | $98M | Launched July 2026; had a sub-account bug in Nov 2026 |
| **Veranta** (ex-Avantis) | Oracle pool on Base | $81M | |
| **GMX v2** | Oracle pool, Arbitrum/Avalanche/MegaETH | $54M | |
| **dYdX v4** | Cosmos app-chain order book | $44M | Shrinking |
| **Nado** (ex-Vertex) | Order book on Ink | $39M | |
| **Paradex** | App-chain | $13M perps, $109M options | Rolled back its chain in Jan 2026 after a pricing glitch |
| **Ostium** | RWA pool on Arbitrum | $10M | Exploited July 2026 |
| **Synthetix** | Perps on Ethereum mainnet | under $1M | |

Sources:
- Hyperliquid HIP-3: https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-3-builder-deployed-perpetuals · https://www.coinperps.com/learn/hyperliquid-hip-3-statistics
- Lighter: https://blockworks.com/news/lighter-opens-public-mainnet
- Arcus: https://thedefiant.io/news/defi/dydx-launches-arcus-dex-stock-tokens-perpetuals-robinhood-chain
- Paradex rollback: https://www.theblock.co/news/markets/2026-01-19-paradexs-free-bitcoin-pricing-glitch-triggers-mass-liquidations-forces-rollback-386128
- Nado: https://www.theblock.co/post/361570/vertex-to-sunset-vrtx-token-move-perp-dex-onto-kraken-backed-ink-layer-2

### What the global table tells you

- **Order books won.** Every venue above $300M OI is an order book or RFQ. Pool venues are an order of magnitude smaller.
- **Purpose-built execution won.** The leaders run their own chain, rollup or off-chain matcher.
- **One venue is 40% of the market**, and its builder-deployed layer is another quarter.
- **Solana in total is small** — the whole table above sums to roughly $300M OI against $16B globally.

That last point cuts both ways: little incumbent liquidity to fight, and little evidence yet that perp traders want to be on Solana for majors. It is one more reason to aim at a segment the leaders do not serve.

## How to study a venue

For each one you look at, write down:

1. Who is the counterparty to a market order?
2. What is the mark price formula?
3. What is the liquidation waterfall, in order?
4. What can admin keys change, and with what delay?
5. What happened in its worst incident?

Hyperliquid, Phoenix and Jupiter have documentation good enough to answer all five. Links are in file 11.
