# 04 — Risk engines, liquidation and insolvency

The matching engine decides who trades. The risk engine decides whether the exchange survives. This is where a perp venue is won or lost, and where most of your design time should go.

## What a risk engine must guarantee

1. Nobody opens a position they cannot collateralise (initial margin check).
2. Positions that fall below maintenance margin are closed before they go negative.
3. When (2) fails, the loss is allocated by a rule fixed in advance.
4. Nobody withdraws value that does not exist.

Points 3 and 4 are the hard ones. A venue that only handles 1 and 2 works until the first fast market.

## The liquidation waterfall

When an account drops below maintenance margin, venues escalate through roughly these stages:

| Stage | What happens | Who pays |
|---|---|---|
| 1. Market liquidation | Position is sold into the book, or a liquidator takes it over at a discount | The liquidated trader (penalty) |
| 2. Backstop | A protocol vault takes the position at mark and manages it | Vault depositors, if it moves against them |
| 3. Insurance fund | Fund covers any negative equity | Past fee payers |
| 4. Socialised loss | Deficit is spread over winners or all users | Everyone, pro rata |
| 5. Auto-deleveraging | Profitable opposing positions are force-closed at the bankruptcy price | Specific winners chosen by a queue |

Design choices inside this waterfall:

- **Partial vs full liquidation.** Closing just enough to restore health is kinder and moves the market less, but needs several transactions exactly when blockspace is scarce.
- **Who liquidates.** Open competition (any bot, paid a fee) is decentralised and fast in calm markets; it becomes a priority-fee auction in a crash. A designated backstop is reliable but concentrates risk.
- **Liquidation price.** Liquidating at mark with a fixed fee is predictable. Liquidating into the book gives a real price but can cascade.
- **Fee destination.** Liquidator, insurance fund, or LPs. Paying the insurance fund builds the buffer that prevents stage 4 and 5.

## Auto-deleveraging and why people hate it

When bankrupt positions cannot be closed in the market and the insurance fund is empty, the venue cancels them against profitable traders on the other side. The classic rule (BitMEX, then nearly everyone) ranks winners by profit and leverage and closes the top of the queue first.

Problems with the queue:

- **It singles people out.** Two traders with the same position get different outcomes based on ranking.
- **It breaks hedges.** A market maker short the perp against a long elsewhere gets its short closed and is left with naked exposure in a crash.
- **It closes at a price the winner did not choose**, usually the bankrupt account's bankruptcy price, forfeiting further profit.
- **It is discretionary in practice.** Thresholds, vault involvement and ordering are venue policy.

The 10–11 October 2025 liquidation cascade made this a mainstream design debate: several venues ran ADL at scale and the fairness of queue-based allocation came under public scrutiny. Tarun Chitra's paper *Autodeleveraging: Impossibilities and Optimization* (arXiv:2512.01112) is the formal treatment; file 11 has the link. File 09 covers the incident.

The alternative family is **pro-rata**: scale every position on the winning side by the same factor, or pay every winner the same fraction of profit. No queue, no ordering advantage. Percolator (file 05) is the most developed open-source version of this idea.

## Lazy accounting: the trick that makes it fit on-chain

You cannot loop over every account to apply funding or socialise a loss — compute limits forbid it. The standard pattern is a **cumulative index**:

```
global:   F  (cumulative funding per unit of position)
account:  f_snapshot  (value of F when the account was last touched)

on touch: owed = position × (F − f_snapshot);  f_snapshot = F
```

One global write updates everyone; each account settles its share in O(1) when next touched. The same pattern handles:

- funding (every venue),
- socialised loss per unit of position,
- pro-rata position scaling (Percolator's `A` index).

If you understand this pattern you can read any on-chain perp codebase.

## Mark price and oracle defences

Most perp exploits are price-input exploits. The defences, roughly in order of value:

| Defence | What it stops |
|---|---|
| **OI caps per market** | Bounds the maximum payout of any manipulation |
| **Staleness and confidence checks** | Trading on an old or uncertain oracle price |
| **Price bands / per-slot move caps** | A single update repricing the whole book |
| **Mark smoothing (TWAP/EMA, median of sources)** | Short spikes triggering liquidations |
| **Profit warmup / withdrawal delay** | Extracting manipulated profit before the price reverts |
| **Collateral weights below 1 for volatile assets** | Borrowing against pumped collateral (the Mango pattern) |
| **Margin scaled to position size and liquidity** | Large positions in thin markets at max leverage |
| **Isolation of risky markets** | One bad market draining the whole venue |
| **Two-step execution** | Oracle latency arbitrage on pool venues |

The canonical attack to keep in mind: an attacker opens a large long and a large short from two accounts, pumps thin spot so the oracle rises, then withdraws or borrows against the winning account's paper profit. The losing account is abandoned. Every defence above attacks one step of that sequence.

## Insurance funds

- Funded by a share of trading fees, liquidation penalties, or staked deposits.
- **Size it against OI, not volume.** The relevant question is how much bad debt one gap move at current OI can create.
- **Who can withdraw matters.** An admin-withdrawable insurance fund is a custody risk, not insurance.
- **Staked insurance** (users deposit for yield and take first loss) grows the fund but creates a run risk if stakers can exit ahead of a loss. Cooldowns exist for this reason.

## A checklist for your own engine

- [ ] Every margin check rounds against the account.
- [ ] Unrealised profit cannot be withdrawn without a rule for whether it is backed.
- [ ] Bad debt has a deterministic owner, decided before it happens.
- [ ] No instruction scans all accounts.
- [ ] Every state that needs advancing has a permissionless crank.
- [ ] Oracle price is checked for age, confidence and maximum move.
- [ ] OI cap exists per market and is sized to liquidity and backstop capital.
- [ ] A user with no open position can always withdraw their deposit.
- [ ] The system state after any failed instruction is unchanged.
- [ ] There is a written answer to "what happens if keepers stop for an hour".

## Where to go deeper

- File 05 for a complete pro-rata engine with proofs.
- File 09 for what went wrong at real venues.
- File 11 for papers: the ADL paper, BitMEX's original insurance-fund and ADL docs, and venue risk documentation.
