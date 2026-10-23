# 05 — Percolator: upstream, the dcccrypto fork, and what you can build on them

Facts here come from reading both repositories on 2026-11-12. Both move daily, so re-check anything you depend on.

## One-paragraph answer

**Percolator** is Anatoly Yakovenko's open-source risk engine for perpetual futures on Solana. It is a Rust library, not an exchange: it holds the accounting rules for margin, PnL, funding, liquidation and insolvency, and is checked with formal proofs. **`aeyakovenko/percolator`** is the original. **`dcccrypto/percolator`** is a fork maintained as the engine underneath a product called Percolator Launch — a "pump.fun for perps" that lets anyone deploy a perp market for any Solana token. The fork tracks upstream by porting commits selectively and adds its own tests, threat model and product hooks. Both say, in capitals, that they are unaudited and not for real funds.

## Upstream: `aeyakovenko/percolator`

| | |
|---|---|
| What | `no_std` Rust library, zero-copy, Apache-2.0 |
| Size | ~23,000 lines in `src/v16.rs`, plus ~20,000 lines of proofs |
| History | 1,885 commits since 19 Oct 2025; last commit 10 Nov 2026 |
| Spec | `spec.md` v16.9.1, "Realizable Full Shared Cross-Margin" (128 KB, normative) |
| Verification | 362 Kani proof harnesses + 61 function-contract proofs, plus fuzz and reference-model tests |
| Status line | "Educational research project. Not production ready. Not audited." |

The first commits in October 2025 were a sharded order-book design ("Router and Slab programs"). It has since been rewritten into a pure risk engine; matching was moved out entirely.

### The problem it sets out to solve

A stressed exchange has profit on paper that the losers cannot pay. Traditional venues resolve this with an ADL queue (file 04). Percolator's position is that profit should never have been treated as money in the first place:

> Capital is senior. Profit is junior. Nobody can withdraw more than the balance sheet holds.

### The three original mechanisms

These are explained most clearly in the fork's README, which keeps the earlier explanatory text.

**1. `h`, the haircut ratio — backed exits**

```
Residual = max(0, V − C_tot − I − …senior claims)
h        = min(Residual, matured_profit_total) / matured_profit_total
```

Every winner can realise `h ×` their matured profit. Healthy system: `h = 1`. Stressed: `h < 1` for everyone equally. Deposits are never haircut. New profit sits in a **warmup reserve** before it counts as matured, so an attacker who spikes a price cannot withdraw the gain before it reverts.

**2. `A / K / F` — pro-rata overhang clearing instead of an ADL queue**

Three global indices per side:

- `A` scales every position on a side by the same factor (used when bankrupt size is removed from open interest).
- `K` accumulates mark-price moves and socialised deficit per unit.
- `F` accumulates funding.

Each account stores snapshots and settles in O(1) when touched (the lazy-index pattern from file 04). No account is singled out. When `A` shrinks too far for precision, the side goes `DrainOnly → ResetPending → Normal` and reopens cleanly with no admin action.

**3. Price/funding envelope — bounded cranks**

A crank may only move the engine's price by `max_price_move_bps_per_slot × elapsed slots`. A large oracle jump is fed in as a staircase. This turns "keepers must be fast enough" into a hard, provable bound on loss per step.

### What v16 added

The current spec goes further than a single global `h`:

- **Multi-asset cross-margin.** One market group holds asset 0 (base) and assets 1..N. An account can hold legs in several and they support each other, giving Hyperliquid-style cross-margin.
- **Source domains.** Each `(asset, side)` is a domain. Profit earned from a domain is usable only up to what that domain's losing side can actually pay:

  ```
  usable_profit_from_domain ≤ realizable_backing_reserved_for_that_domain
  ```

  If someone manipulates asset A, their paper profit on A cannot become buying power on asset B. This is the direct answer to the Mango-style attack.
- **Liens.** Any durable use of profit — withdrawing it, opening new risk against it — must reserve specific backing, so the same dollar of backing is never promised twice.
- **Backing providers.** LPs post capital to back a domain's winners and earn utilisation fees.
- **Per-domain bankruptcy.** Bad debt stays in the domain that produced it. There is no global loss pool.
- **Account-local progress.** No instruction scans all accounts. A crank touches one bounded portfolio.

### What the engine does not do

By design the library has no matching, no oracle parsing, no token transfers and no signature checks. Those belong to the **wrapper**. The repo's own summary of the trusted base: Solana's rollback on failure, wrapper correctness, and the bounds of what Kani can prove.

### The upstream family

| Repo | Role |
|---|---|
| `aeyakovenko/percolator` | The risk engine library |
| `aeyakovenko/percolator-prog` | The Solana program wrapping it: market slab, portfolios, `TradeCpi` / `TradeNoCpi`, oracle and authority model. Most active (pushed 11 Nov 2026) |
| `aeyakovenko/percolator-match` | Reference matcher: passive LP quoting ±50 bps around oracle |
| `aeyakovenko/percolator-cli` | Command-line client |
| `aeyakovenko/percolator-meta` | Governance bootstrap and reward system (insurance deposits, COIN allocation voting, Squads and MetaDAO integration) |
| `aeyakovenko/percolator-stress-test` | Load testing |

`percolator-prog` is where the product shape is visible:

- A market is **one slab account holding up to 5,834 assets**; per-trade compute does not grow with the number of assets.
- **Assets 1..N can be created permissionlessly for a fee**, and the fee goes to asset-0 insurance.
- Each asset has its own insurance, backing and admin key; the admin key can be burned.
- A faulty or malicious asset cannot drain another asset's insurance, backing or winners — each claim has a named test.
- Trading goes through a **matcher CPI**: the LP registers a matcher program, the wrapper calls it, and validates the returned price and size as untrusted input.

That last point is the architectural idea worth stealing even if you never use the code: **a verified risk core with pluggable, untrusted pricing**.

## The fork: `dcccrypto/percolator`

| | |
|---|---|
| Relationship | GitHub fork; default branch `main`; **605 commits ahead, 935 behind** upstream `master` |
| Diverged from | Common ancestor dated 25 Apr 2026 |
| Spec | v16.8.3 (upstream is v16.9.1) |
| `main` last commit | 28 Sep 2026; other branches pushed 12 Nov 2026 |
| Verification | 325 Kani harnesses, 0 function-contract proofs, Kani pinned to 0.67.0, CI workflow |

### How it differs from upstream

| Area | Upstream | Fork |
|---|---|---|
| Purpose | Research engine | Engine for a shipped product stack |
| Sync model | Source of truth | Ports upstream commits one at a time ("sync: port …", "adopt upstream …") rather than merging |
| Spec version | v16.9.1 | v16.8.3 — behind on the latest settlement-ordering and close-serialisation changes |
| Proof layers | Plain proofs + function contracts + inductive closure layer | Plain proofs only; five upstream contract properties re-expressed as plain proofs |
| Extra tests | — | Regression suites for its own audit findings (F-02, F-03, F-04, I2, K, Q, R, issue numbers 133/143/448/457), LP-vault tests, griefing-economics tests |
| Extra code | — | `fork-facade` feature exposing extra entry points for its wrapper; LP vault hooks; `proofs_v17_fork.rs` |
| Docs | Terse README + spec | Long explanatory README, `THREAT_MODEL.md`, `docs/v17` convergence notes |
| CI | Shell scripts for long Kani runs | GitHub Actions including an `audit-scan` job |

Things to know before leaning on the fork:

- **It lags.** Upstream landed five fixes on 10 Nov 2026 alone. Any fix not yet ported is a known gap in the fork.
- **Its `THREAT_MODEL.md` is stale** — dated April 2026 against spec v12.17 — and records an externally-owned admin key controlling upgrades and config, to be moved to a multisig before production.
- **The convergence docs are written as instructions to an autonomous coding agent.** Much of the porting appears to be agent-driven. That is not disqualifying, but it means you should weigh the test evidence, not the commit count.
- **A sibling repo publishes a disclosure against the upstream mainnet bounty program** (`dcccrypto/percolatorbounty`), describing a conditional finding that needed a ≥20% SOL move within one 60-second oracle window. It shows the team reads the code closely; it also shows real findings exist.

### The fork's product stack

The engine fork is one piece of about twenty repositories:

| Repo | What it is |
|---|---|
| `percolator-launch` | The product. Next.js app + API + keeper + indexer. "Launch perpetual futures markets for any Solana token in one click." README shows it live on devnet |
| `percolator-prog` (fork) | Wrapper program |
| `percolator-match` | Passive LP + vAMM matcher with skew-aware spread |
| `percolator-stake` | Insurance LP staking |
| `percolator-vault` | LP vault |
| `percolator-portfolio` | USDC cross-margin wrapper across isolated markets |
| `percolator-nft` | Positions as Token-2022 NFTs |
| `percolator-keeper`, `-indexer`, `-oracle-keeper`, `-sdk`, `-mobile` | Operations and clients |
| `percolator-perp-liquidity` | Research: permissionless perps for long-tail tokens |

Design choices in `percolator-launch` as its README describes them:

- **Coin-margined.** You post BONK to trade the BONK perp; PnL is in BONK. This avoids needing USDC liquidity per market but makes collateral value fall with the price.
- **Oracle from DEX pools** (PumpSwap, Raydium, Meteora parsers) or pushed by a keeper with a per-slot price cap.
- **vAMM for initial liquidity**, insurance fund fed by fees.
- **Slab tiers** of 256 / 1,024 / 4,096 accounts, costing roughly 0.44 / 1.73 / 6.87 SOL in rent.

### Read `percolator-perp-liquidity` closely

This is the most useful document in the whole ecosystem for your stated interest. It is the team's own analysis of launching perps on tokens with no liquidity, and it is candid. Its repo description ends: *"fee engine validated; solvency unsafe-as-specified (research-stage)."*

Its core argument:

> When leveraged longs win, someone pays them. On a one-sided book the money can only come from four places: real opposing traders, a vault/LP, the creator, or the token's own spot market. There is no fifth source.

Its proposed design: match peer-to-peer first; a **bounded, paid seed** takes the residual one-sided flow; the OI cap is sized so the seed's worst loss equals the seed; winners are paid parimutuel-style with a `credit_rate = min(1, backing / claim)` haircut.

Its own list of unsolved problems:

- Routing a taker order against a program-owned seed atomically.
- Fresh-token oracles can be manipulated for less than the seed is worth.
- A keeper is still a central point.
- Nobody has shown a profitable permissionless one-sided memecoin leverage product.

Those four bullets are a ready-made problem list. File 10 builds on them.

## What you can build with these

Ordered roughly from least to most original engineering.

1. **A matcher.** Write a pricing program that plugs into `TradeCpi`: an order book, a skew-aware vAMM, an oracle-plus-spread quoter that hedges elsewhere. Smallest surface, real differentiation. Start from `percolator-match`.
2. **Keeper and liquidation infrastructure.** Cranks, oracle pushers, liquidators. Every Percolator deployment needs them.
3. **A front-end or aggregator** over existing Percolator markets.
4. **Backing-provider and insurance vaults.** Products that let passive capital back specific domains for yield, with honest risk disclosure. The deterministic reward counters in `percolator-prog` are designed for this.
5. **A cross-margin or portfolio layer** across markets (see `percolator-portfolio`).
6. **Your own wrapper** around the upstream engine with a different product thesis: different oracle policy, collateral, listing rules, fee routing.
7. **A long-tail listing mechanism** that actually answers the four open problems above.
8. **Position tokenisation and structured products** on top of positions.
9. **Your own engine, borrowing the ideas.** Junior profit, source-domain caps, lazy indices and bounded cranks are ideas, not code. If the v16 spec is heavier than you need, a smaller engine with the same invariants and your own proofs is a legitimate path.

## Cautions

- **Neither codebase is audited.** Upstream fixed several value-accounting bugs in the week this was written. Treat it as a moving research target.
- **The spec is hard.** 128 KB of normative text with a dense vocabulary. Budget days, not hours.
- **Formal proofs cover the engine, not the wrapper.** Oracle handling, authority checks and token custody are where a deployment will actually break, and they are outside the proofs.
- **Pro-rata haircuts have a UX cost.** A trader who is paid 92% of a win will be unhappy even if the alternative venue would have ADL'd them. This is a product problem as much as an engineering one.
- **Check the name.** "Percolator" has attracted tokens and sites unconnected to the upstream author. Verify anything claiming affiliation against the GitHub repos. File 09 notes what could be confirmed.

## How to read the code

1. Fork README — the `h`, `A/K/F` and envelope explanation, in plain language.
2. Upstream `spec.md`, section 0 only — the 37 "non-negotiable requirements".
3. `aeyakovenko/percolator-prog` README — product specification and trust boundaries.
4. `percolator-match` — small enough to read in full.
5. `percolator-perp-liquidity` README and `model/`.
6. Then `src/v16.rs`, guided by `tests/v16_spec_tests.rs`. Read the tests named `v16_attack_*` in `percolator-prog` first: each is one attack and its defence.
