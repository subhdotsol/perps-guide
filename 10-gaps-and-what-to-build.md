# 10 — Gaps and what to build

Researched 2026-11-12. Rankings are my judgement from the evidence in files 05, 06 and 09; the evidence is cited there.

## Your example: new tokens have no perp market

You are right that this is open. As of November 2026 nobody runs a liquid, safe perp on a freshly launched memecoin. Here is who tried.

| Attempt | Mechanism | What limited it |
|---|---|---|
| **Hyperliquid HIP-3** | Stake 500k HYPE, deploy your own perp DEX with your own oracle | Capital gate far too high for long tail. One deployer has over 95% of volume, and it lists equities, not memecoins. Most paid listings never earned back their auction cost |
| **Hyperliquid hyperps** | No spot oracle; mark is an average of the perp's own price | Curated, isolated-only, low leverage |
| **Percolator Launch** (dcccrypto) | Any SPL token, coin-margined, vAMM seed, DEX-pool oracle | Unaudited; failed a third-party review in Apr 2026; DEX-pool oracle is the Mango-style surface |
| **percolator.trade** | Percolator-based; pinned Raydium pool average for unlisted tokens | Mainnet in "lab mode" with first-party capital only, gated on an audit |
| **Arqen** (Solana, Sep 2026) | 20 memecoin perps, each on an isolated community-funded pool, strict caps | Safe because tiny: one market launched with $50 of backing and a $20 max position |
| **pump.fun perps** (Oct 2026) | Routes to Hyperliquid | Does not list pump.fun's own tokens at all |
| **Jupiter Perps** | Added six curated order-book markets | Curated, not permissionless |
| **Drift** | Had permissionless-leaning listing | The Apr 2026 drain used a fake thin-liquidity listing as collateral |

Sources: https://0xarchive.io/resources/research/addressing-hyperliquid-s-hip-3-deployer-bottleneck · https://solanacompass.com/news/arqen-perps-goes-live-on-solana-with-20-long-tail-markets-and-community-funded-isolated-pools · https://percolator.trade/waitlist · https://github.com/dcccrypto/percolator-perp-liquidity

### Why it is hard

Three problems stack, and you must solve all three:

1. **No counterparty.** Flow on a pumping token is almost all long. Winners must be paid by real shorts, an LP, the creator, or the token's spot market. There is no fifth source.
2. **No trustworthy price.** The only price is a thin DEX pool. If moving it costs less than the profit available from the perp, someone will move it. Averaging over time raises the cost but does not remove it.
3. **Reflexivity.** Liquidating perp positions moves the spot pool that sets the price that triggers more liquidations.

Every attempt above solves one or two by giving up something: Arqen gives up size, HIP-3 gives up permissionlessness, vAMM launchers give up solvency guarantees.

### Design directions worth exploring

- **OI capped by spot depth, enforced on-chain.** Maximum OI as a function of the pool's real liquidity within a price band, recomputed every trade. This is the single most load-bearing control.
- **Bounded, paid seed as counterparty.** The `percolator-perp-liquidity` design: a creator or LP stake takes residual one-sided flow, earns fees and funding for it, and its maximum loss equals its size.
- **Transparent haircut instead of insolvency.** Pay winners `min(1, backing / claims)`. Honest, provably solvent, and a harder sell to traders.
- **Coin-margined markets.** Collateral and PnL in the token itself, so losses cannot leak into stablecoin pools. Cost: collateral value collapses exactly when longs are losing.
- **Leverage that falls as the book gets one-sided.** Generous when empty, tight when crowded.
- **Graduation.** Start capped and isolated; unlock size and leverage as the token gains spot depth, holders and time.
- **Hedge into spot.** The LP offsets perp skew by buying or selling the spot token. Works only while the pool is deep enough, which brings you back to the OI cap.
- **Bonding-curve-native perps.** Before a token leaves its launch curve, the curve itself is a deterministic price and liquidity source. A perp settled against the curve avoids the thin-pool oracle entirely for that phase. I found nobody doing this; that could mean opportunity or a flaw I have not seen.

If you pursue this, write the attacker's profit-and-loss before you write code: cost to move the price by X% for the averaging window, versus maximum extractable from the perp at the OI cap. The design is safe only where the first exceeds the second with margin.

## Ranked gaps

Scored on how open the problem is and whether a small team can plausibly ship.

| # | Gap | Why still open | Hardest part |
|---|---|---|---|
| 1 | **Manipulation-resistant perps for new and long-tail tokens** | Every attempt is curated, capital-gated, toy-sized or unaudited | Mark price and OI cap that track real exit liquidity and survive many-wallet attacks |
| 2 | **Oracle and risk layer for assets with no CEX listing**, sold to launchers | Each long-tail project rolled its own and several were exploited | Thin pools are cheap to move over any averaging window |
| 3 | **Admin-minimised venue** — immutable or timelocked, rate-limited withdrawals, liquidity-gated listings | Solana's largest perp venue lost ~$285M to governance, not code | Tuning risk and responding to incidents without keys |
| 4 | **Audited production wrapper and matcher for a Percolator-style engine** | Upstream is explicitly educational; forks failed review | Proofs do not cover wrapper, oracle or matcher; someone must still quote |
| 5 | **Liquidity bootstrapping for new perp markets** — isolated bounded-loss vaults, or a matcher competition | Arqen opened with $50 depth; most HIP-3 listings do not pay back | LPs lose to informed memecoin flow |
| 6 | **Fair, published loss-socialisation as a selling point** | Open since Oct 2025 and now formalised in research | Traders only value it after a crash |
| 7 | **Coin-margined memecoin perps** | Only on devnet | Collateral and exposure are perfectly correlated |
| 8 | **Private positions on Solana** | Existing private perps are off-Solana | Liquidators and the engine need the data you are hiding |
| 9 | **Cross-venue aggregator / unified margin** | Ranger shut down; the slot is empty | Ranger's failure suggests weak economics; real cross-margin needs venue cooperation |
| 10 | **Funding-rate and basis products for Solana venues** | Boros exists elsewhere; nothing found on Solana | Thin demand; settlement integrity of funding prints |
| 11 | **Embedded perps for wallets, launchpads, apps** | Distribution partners are choosing between a few backends | No moat; fee share gets competed away |
| 12 | **Growth without points** | Post-airdrop volume collapse is now normal | Bootstrapping liquidity with no incentives in a market where OI has halved |

Areas I could not find good evidence on, so they are unranked: options and structured hedges on Solana perps, and vault or strategy products built on top of perps.

## Where not to compete

- **Majors (BTC, ETH, SOL).** Hyperliquid, Pacifica, Jupiter, Phoenix and Velocity are all there, and US onshore venues now are too.
- **Equity and RWA perps.** One deployer dominates on Hyperliquid, and regulated US venues were cleared for single-stock perps this month.
- **A general-purpose order-book venue.** Needs market-maker relationships and capital that a small team does not have on day one.

## My recommendation

Gaps 1, 2, 4 and 5 are the same product seen from four sides, and they line up with your instinct. A coherent thesis:

> A permissionless perp launcher for new Solana tokens whose safety comes from on-chain liquidity-scaled OI caps and bounded per-market loss, built as a wrapper and matcher over a Percolator-style engine, with no admin key able to change listings or limits.

Why this and not something else:

- It targets the one segment the large venues have each declined to serve.
- The upstream engine already provides per-asset isolation and permissionless asset creation, so you are not writing a risk engine from scratch.
- Gap 3 comes almost for free if you design for it from the start, and after Drift it is a selling point.

What would make me change this recommendation: if your attacker P&L analysis shows the safe OI cap on a typical new token is too small to generate meaningful fees. Arqen's $20 position limit suggests that outcome is possible. Do that analysis in week one, on real pool data from 20 recent launches, before committing.

A lower-risk first step that still builds toward it: ship **gap 2 alone** — a liquidity-aware price and OI-cap module with a published manipulation-cost model — and offer it to the existing Percolator launchers. It is small, testable against historical pool data, and tells you whether the bigger product is viable.

## Questions only you can answer

- Venue or infrastructure? Running a venue means custody risk, keepers, incident response and regulatory exposure. Selling a component to venues means none of those and a smaller upside.
- Who provides the first dollar of counterparty capital in each market, and what are they paid?
- Would your users accept a visible haircut on wins in exchange for access to markets that exist nowhere else?
- Which jurisdictions will you serve, and have you had that conversation with a lawyer?
