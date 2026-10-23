# 01 — Perp fundamentals

Read this first. Everything later assumes you can do the arithmetic here in your head.

## What a perpetual future is

A perp is a contract that pays you `size × (exit price − entry price)` with no expiry date. Nobody delivers the underlying asset. Two things keep the contract price near the real ("spot" or "index") price:

1. **Funding** — a periodic payment between longs and shorts. When the perp trades above spot, longs pay shorts, which makes being long expensive and pulls the price down. Below spot, shorts pay longs.
2. **Arbitrage** — traders who buy spot and short the perp (or the reverse) collect funding until the gap closes.

A perp exchange is therefore three machines bolted together:

| Machine | Job | If it fails |
|---|---|---|
| Matching | Finds a counterparty and a price for each trade | No liquidity, bad fills |
| Risk engine | Decides who may open, who must be liquidated, who gets paid | Bad debt, insolvency |
| Price inputs (oracle, mark, funding) | Tells the risk engine what positions are worth | Manipulation, unfair liquidations |

Most of the engineering difficulty is in the second and third rows, not the first.

## The three prices

| Price | What it is | Used for |
|---|---|---|
| **Index / oracle price** | Spot price aggregated from external venues | Funding, and on many DEXs liquidation |
| **Mark price** | The exchange's "fair" price of the perp, usually a smoothed blend of index and the perp's own book | Unrealised PnL, margin checks, liquidation |
| **Last / execution price** | The price a trade actually filled at | Realised PnL |

Liquidating on last price lets one large order trigger a cascade, so every serious venue liquidates on mark. How mark is built is a core design decision: the more it listens to its own order book, the easier it is to manipulate; the more it listens to an external oracle, the more you depend on that oracle existing and being honest.

## Margin and leverage

- **Notional** = `size × price`.
- **Initial margin (IM)** = collateral required to open. `IM fraction = 1 / max leverage`. 20× means 5%.
- **Maintenance margin (MM)** = collateral required to stay open. Usually about half of IM.
- **Equity** = collateral + unrealised PnL − fees and funding owed.
- **Liquidatable** when `equity < MM × notional`.

Margin modes:

| Mode | Collateral pool | Trade-off |
|---|---|---|
| **Isolated** | One position, one bucket | A loss cannot exceed that bucket; capital-inefficient |
| **Cross** | All positions share one bucket | Winners support losers; one bad position can sink the account |
| **Portfolio margin** | Margin computed on net risk across correlated positions | Most efficient; hardest to get right |

Contract types:

| Type | Collateral | PnL paid in | Note |
|---|---|---|---|
| **Linear** (USD-margined) | Stablecoin | Stablecoin | The standard today |
| **Inverse** (coin-margined) | The traded coin | The traded coin | Collateral falls in value as a long loses — doubly exposed |
| **Quanto** | A third asset | That third asset | Adds correlation risk between collateral and underlying |

## Liquidation price

For an isolated position opened at entry `E` with leverage `L` and maintenance fraction `m`, ignoring fees and funding:

```
long:   P_liq = E × (1 − 1/L) / (1 − m)
short:  P_liq = E × (1 + 1/L) / (1 + m)
```

Worked example: long 1 SOL at $100, 10× leverage, `m` = 2.5%.

- Collateral posted: $10.
- `P_liq = 100 × 0.9 / 0.975 = $92.31`.
- At $92.31 equity is `10 − 7.69 = $2.31`, and MM is `0.025 × 92.31 = $2.31`. They meet, so liquidation triggers.
- **Bankruptcy price** is where equity hits zero: $90. The $2.31 between liquidation and bankruptcy is the buffer that pays the liquidator and absorbs slippage. If the position cannot be closed before $90, the shortfall is **bad debt**.

Everything in file 04 is about what happens when that buffer is not enough.

## Funding

The textbook formula (BitMEX, and most CEXs since):

```
premium      = (perp price − index price) / index price      (time-averaged)
funding rate = premium + clamp(interest − premium, −0.05%, +0.05%)
payment      = position notional × funding rate
```

`interest` is a fixed small number (commonly 0.01% per 8 hours). The clamp means the rate sits at the interest rate while the premium is small and tracks the premium once it grows.

Venues differ in three knobs — check each venue's docs rather than trusting a summary:

- **Interval** — 8 hours on most CEXs; hourly or continuous on most DEXs.
- **Cap** — the maximum rate per interval. A tight cap protects traders but lets the perp drift far from spot on a one-sided market.
- **Reference** — premium measured against mark, mid-price, or an impact price (the price to fill a fixed notional).

Funding is zero-sum between traders on a matched book. On a pool-based venue (file 03) the pool is the counterparty, so the "funding" is usually replaced by a **borrow fee** paid to the pool.

## Open interest and why it matters

**Open interest (OI)** is the total size of open contracts. On an order book, long OI always equals short OI. On a pool venue it does not: the pool holds the difference, called **skew**.

OI is the number that bounds your risk. Every oracle-manipulation attack on a perp venue is some version of "build OI larger than the cost of moving the price". **OI caps** per market are the bluntest and most effective defence.

## The lifecycle of one trade

1. Trader deposits collateral into a vault the program controls.
2. Trader submits an order. The matching layer finds a counterparty and price.
3. Risk engine checks post-trade equity ≥ IM. If not, reject.
4. Position is recorded. Fees go to the protocol, LPs or insurance fund.
5. Each funding interval, funding is accrued (usually lazily, settled when the account is next touched).
6. Price moves. Mark price updates. Unrealised PnL changes.
7. Either the trader closes (PnL is realised into collateral), or equity drops below MM and a liquidator closes the position for them.
8. Trader withdraws. The engine must check the withdrawal does not leave remaining positions under-margined — and, on a well-designed engine, that the profit being withdrawn actually exists (file 05).

## Check yourself

- Why does a 50× position get liquidated on a 1.5% move rather than a 2% move?
- A market has 90% of OI long and no natural shorts. What happens to funding? Who is on the other side on an order book? On a pool?
- You hold a winning long. The losing shorts go bankrupt faster than they can be liquidated. Where does your profit come from?

If the last one is not obvious, that is the question files 04 and 05 answer.
