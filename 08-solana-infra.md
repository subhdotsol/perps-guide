# 08 — Solana infrastructure for perps

State of 2026-11-12. Each item is marked **live**, **partly live** or **proposed**. Most dates and adoption figures here come from press coverage of the primary announcements.

## Oracles

Your oracle choice is the most consequential infrastructure decision you will make.

| Oracle | Status | What to know |
|---|---|---|
| **Pyth Core** | Live, now paid | Since 26 Aug 2026 the price service and pusher need an API key, with plans announced from $500/month. On-chain interfaces and addresses unchanged. Pull model: you fetch a signed update and submit it in your transaction. Each price carries a confidence interval — use it |
| **Pyth Pro** (formerly Lazer) | Live | Low-latency product for venues that need sub-slot updates |
| **Chainlink Data Streams** | Live | On Solana mainnet since Feb 2025. Reported users include Jupiter Perps, Kamino and GMTrade |
| **Edge / Doves** (Chaos Labs with Jupiter) | Live | Jupiter Perps' primary oracle |
| **Switchboard** | **Shut down** | Support ended 25 Sep 2026 on all chains. Any tutorial or codebase using it is out of date |
| **Stork** | Live | Docs exist; no evidence found of which Solana perp venues use it |
| **DEX pool price** | Always available | The only source for unlisted tokens. Manipulable in proportion to pool depth (files 09, 10) |

Sources: https://docs.pyth.network/price-feeds/core/upgrade · https://docs.pyth.network/pro · https://docs.chain.link/data-streams/release-notes · https://docs.jup.ag/user-docs/trade/perps/technical-reference · https://solanacompass.com/news/switchboard-oracle-protocol-shuts-down-giving-solana-defi-six-days-to-migrate

### What this means for you

- **Budget for oracle cost** from day one; free standard feeds are gone.
- **Use more than one source.** Switchboard's users had six days to migrate. Jupiter's primary-plus-two-checks pattern is the model.
- **For long-tail tokens no oracle network will help you.** You will be building your own price from pool state, which is why file 10 ranks a liquidity-aware oracle layer as a product in itself.

### Minimum checks on any price you accept

- Age below a hard limit.
- Confidence interval below a fraction of price.
- Deviation from your last accepted price bounded per unit time.
- Disagreement between sources triggers reduce-only mode, not a best guess.

## Consensus and networking

| Item | Status | Relevance |
|---|---|---|
| **Alpenglow** | Live on mainnet since 21 Oct 2026 | New consensus aimed at much faster finality. The headline 150 ms target is not independently confirmed; one outlet reported a far slower reading at launch. Measure it yourself before assuming it in your liquidation timing |
| **Rotor** (new block propagation) | Not scheduled | The old propagation layer remains |
| **DoubleZero** | Live | Dedicated fibre network for validators; roughly 60% of stake connected by Q3 2026 |

Sources: https://solana.com/upgrades/alpenglow · https://cointelegraph.com/news/solanas-alpenglow-launches-mainnet-99-faster-finality · https://solanacompass.com/news/doublezero-posts-217b-total-connected-value-and-59-solana-mainnet-stake-weight-in-q2-2026

## Block building and transaction ordering

This is where on-chain order books live or die. The core issue: if takers can always land ahead of a maker's cancel, makers quote wide.

| Item | Status | What it does |
|---|---|---|
| **Jito BAM** (Block Assembly Marketplace) | Live; roughly 28–37% of stake during 2026 | Programmable block building. Plugins let an application define ordering rules, such as processing cancels before takes. First third-party node operators named Nov 2026 |
| **Application-controlled execution (ACE)** | Partly live, through BAM plugins | The app, not the validator, sets sequencing for its own transactions. Detail on which plugins are in production was hard to find |
| **Multiple concurrent proposers** ("Constellation") | Proposed | Several validators propose in the same short window, removing any single leader's power to censor or reorder. No timeline |
| **Internet Capital Markets roadmap** | Roadmap (Jul 2025) | Names market microstructure as Solana's priority: BAM near term, concurrent proposers and protocol-level ACE running into 2027 |

Sources: https://www.helius.dev/blog/block-assembly-marketplace-bam · https://constellation.anza.xyz/ · https://www.anza.xyz/blog/the-internet-capital-markets-roadmap · https://solanacompass.com/news/jito-names-triton-one-bitwise-and-sol-strategies-as-first-third-party-bam-node-operators

**Practical reading:** do not design around features that are only proposed. If your design needs cancel priority today, your options are a BAM plugin, an in-program speed bump, or matching off the base layer.

## Execution patterns

| Pattern | Status | Relevance |
|---|---|---|
| **Proprietary AMMs** | Live; roughly 60–80% of Solana DEX volume in 2026 | Market makers run private on-chain pricing programs updated cheaply each slot. This is the spot-market precedent for Percolator's matcher model and Phoenix's splines |
| **MagicBlock ephemeral rollups** | Live (validator v1.0 in Oct 2026) | Temporarily move accounts to a fast side environment and commit back. Its flagship perp user, Flash Trade, has closed |
| **Network extensions / app rollups** | Live (Bullet, BULK) | Run matching on your own sequencer or validator set, settle on Solana |

Sources: https://www.helius.dev/blog/solanas-proprietary-amm-revolution · https://solanacompass.com/news/magicblock-releases-validator-v10-for-ephemeral-rollups-on-solana-with-new-session-and-commit-fees

## Where to run matching: the real decision

| Option | Gain | Cost |
|---|---|---|
| Fully on Solana L1 | Composability, no extra trust, simplest custody story | Slot-time latency, compute limits, makers exposed to ordering |
| L1 with BAM plugin | Ordering guarantees on the base chain | Only on the share of stake running BAM |
| Ephemeral rollup | Speed with L1 settlement | Extra trust and operational dependency |
| Own sequencer or network | Full control, CEX-like speed | You operate infrastructure; weaker decentralisation claim |
| No matching at all (oracle pool, or matcher CPI) | Sidesteps the problem | Moves the risk to LPs or to whoever writes the matcher |

For a long-tail launcher the last row is the natural fit: there are no competing market makers to protect, so ordering matters less than price integrity and loss bounds.

## Operational stack you will need regardless

- **RPC with streaming** (Geyser/gRPC) for the indexer and keepers.
- **Indexer + database** for positions, trades, funding history and charts.
- **Keepers** for cranks, oracle updates and liquidations, with monitoring on their balances and lag.
- **Transaction landing strategy**: priority fees, tips, retries, and a plan for congestion.
- **Multisig with a timelock** for any authority you cannot burn. After Drift, a zero-delay multisig is a known anti-pattern.
