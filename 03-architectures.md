# 03 — Perp exchange architectures

There are only a handful of ways to answer "who is on the other side of the trade, and at what price?". Every venue is one of these or a hybrid. Current status of each named venue is in file 06.

## The five matching designs

### 1. Central limit order book (CLOB)

Traders and market makers post limit orders; a matching engine pairs them by price then time.

| Where matching runs | Examples | Trade-off |
|---|---|---|
| Purpose-built chain | Hyperliquid, dYdX v4 | Fast and cheap; you run a chain and its validator set |
| ZK rollup / validium | Lighter, edgeX, Paradex | Off-chain speed with proofs; sequencer is a central point |
| Fully on a general L1 | Phoenix, Mango v4, OpenBook (Solana) | Composable and transparent; limited by block time and compute |
| Off-chain book, on-chain settle | Older dYdX v3, many "hybrid" DEXs | Simple and fast; you must trust the operator's matching |

- **Strengths:** best price discovery, zero-sum between traders (no LP pool to drain), funding works naturally.
- **Weaknesses:** needs professional market makers. A new or long-tail market has an empty book.
- **Hard parts on Solana:** makers must cancel and requote faster than takers can pick them off. Without cancel priority or a speed bump, makers quote wide or leave.

### 2. Virtual AMM (vAMM)

A constant-product curve `x × y = k` with no real tokens in it. It only sets the price; collateral sits in a separate vault.

- **Examples:** Perpetual Protocol v1, Drift v1. Drift v2 keeps an AMM as one of several liquidity sources.
- **Strengths:** always-on liquidity from the first second, no market makers needed.
- **Weaknesses:** the protocol itself is the counterparty to net skew. If longs win and shorts were never there, the vault owes money nobody posted. `k` is hard to set: too low gives terrible slippage, too high gives the protocol large exposure.
- **Why it matters to you:** it is the obvious answer for long-tail tokens and the one that has failed most often.

### 3. Oracle-priced pool (peer-to-pool)

LPs deposit real assets into a pool. Traders open positions at the oracle price with zero slippage; the pool is the counterparty.

- **Examples:** GMX, Jupiter Perps (JLP), Flash Trade, Adrena, Gains, Ostium, Avantis.
- **Strengths:** deep liquidity at oracle price, simple UX, LPs earn fees plus trader losses.
- **Weaknesses:**
  - LPs lose when traders win. Informed or oracle-front-running flow is directly paid by LPs.
  - No price discovery — the venue cannot list anything without a reliable external oracle.
  - Skew is a real exposure, managed by borrow fees, price-impact fees and OI caps.
- **Design levers:** borrow fee curve, price-impact fee, per-asset OI caps, two-step execution (request then keeper fulfils at a later oracle price) to defeat latency arbitrage.

### 4. RFQ and intents

The trader signs "I want to open X"; market makers or solvers compete to fill it, often hedging on a CEX.

- **Examples:** Symmio-based front-ends, several aggregators.
- **Strengths:** CEX-depth pricing on-chain, wide asset coverage.
- **Weaknesses:** counterparty is a bilateral market maker; liveness and solvency depend on them.

### 5. Pluggable matcher

The risk engine does not match at all. Each LP registers its own pricing program; the engine calls it and validates the answer.

- **Example:** Percolator (`TradeCpi` → matcher program). See file 05.
- **Strengths:** separates the part that must be provably correct (risk) from the part that should be competitive (pricing). Order books, vAMMs and prop AMMs can all plug into the same engine.
- **Weaknesses:** new and unproven; someone still has to supply the liquidity.

### Hybrids

Real venues mix these. Drift routes a taker order through a short just-in-time auction for makers, then resting limit orders, then its AMM. Hyperliquid is a CLOB with a protocol vault (HLP) acting as market maker and liquidation backstop. When you study a venue, ask which design handles normal flow and which handles the leftover.

## Comparison

| | CLOB | vAMM | Oracle pool | RFQ | Pluggable matcher |
|---|---|---|---|---|---|
| Needs market makers | Yes | No | No (needs LPs) | Yes | Depends on matcher |
| Who bears skew | Nobody (matched) | Protocol | LP pool | Market maker | LP behind the matcher |
| Price discovery | Yes | Partial | No | No | Depends |
| Works with no oracle | Yes (hyperp) | Yes | No | No | Depends |
| Long-tail listing | Hard | Easy, risky | Hard | Medium | Open question |
| Main failure mode | Empty book | Unbacked profit | LP drained | MM default | Unproven |

## Components every venue has

```
            ┌───────────────┐     ┌──────────────┐
  trader ──►│  Front-end /  │────►│   Matching   │
            │  API / SDK    │     └──────┬───────┘
            └───────────────┘            │ fill (price, size)
                                         ▼
  oracle ──────────────────────►┌──────────────────┐◄──── keepers / cranks
                                │   Risk engine    │      (funding, liquidation,
  vault (collateral custody) ◄─►│ margin, PnL,     │       settlement)
                                │ funding, liq.    │
  insurance fund ◄─────────────►└────────┬─────────┘
                                         │ events
                                         ▼
                                ┌──────────────────┐
                                │ Indexer, history,│
                                │ charts, analytics│
                                └──────────────────┘
```

| Component | On-chain or off | What you must decide |
|---|---|---|
| Vault / custody | On | Which collateral, who can move it, upgrade authority |
| Risk engine | On | Margin model, liquidation rules, insolvency handling |
| Matching | Either | Which of the five designs |
| Oracle adapter | On, fed from off | Which feeds, staleness and confidence limits, fallback |
| Keepers | Off | Who runs them, how they are paid, what happens if they stop |
| Liquidators | Off | Open competition or designated backstop |
| Insurance fund | On | Funded by what, who can withdraw |
| Indexer + API | Off | Needed for any usable UI; not optional |
| Admin / governance | On | Which keys exist and whether they can be burned |

## Solana constraints that shape the design

- **Write-lock contention.** If every trade writes one global market account, trades in that market cannot run in parallel and fees for that account spike under load. Designs that touch only per-user accounts scale better. This is the core motivation for Percolator's "account-local" engine.
- **Compute limit.** 1.4M compute units per transaction. You cannot loop over all accounts in a market. Anything global (funding, ADL, loss socialisation) must be lazy: update one global index, settle each account when it is next touched.
- **Account size.** 10 MiB maximum. A "slab" holding a whole market is feasible but bounded.
- **Transaction size.** 1232 bytes, which limits how many accounts and how much oracle data one trade can carry. Address lookup tables help.
- **No scheduled execution.** Nothing happens unless someone sends a transaction. Funding, liquidations and oracle updates all need keepers, and a design is only as live as its least-reliable crank.
- **Atomic rollback.** A failed instruction reverts everything. Good engines lean on this: check invariants at the end and return an error.
- **Transaction landing.** Liquidations compete for block space exactly when the chain is busiest. Assume some will land late and size the maintenance buffer accordingly.

## Questions to answer before writing code

1. Who is the counterparty on a one-sided market?
2. What is the mark price when the oracle is stale, wrong or absent?
3. When a position goes bankrupt faster than it can be closed, who pays, in what order?
4. What can an admin key do, and can a user always exit without one?
5. What must keepers do, and what breaks if they stop for an hour?
