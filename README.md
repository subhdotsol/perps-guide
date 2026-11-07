# Perps study pack

A reading path for building a perpetual futures exchange on Solana. Compiled 2026-11-12. Venue status, volumes and repository states change quickly — re-check anything you plan to rely on.

## Read in this order

| # | File | What you get | Time |
|---|---|---|---|
| 01 | [Perp fundamentals](01-perp-fundamentals.md) | Prices, margin, funding, liquidation maths | 30 min |
| 02 | [Jargon glossary](02-jargon-glossary.md) | Every term used below; keep open as a reference | skim |
| 03 | [Architectures](03-architectures.md) | The five matching designs, system components, Solana constraints | 40 min |
| 04 | [Risk engines, liquidation, ADL](04-risk-engine-liquidations-adl.md) | Liquidation waterfall, ADL, lazy accounting, oracle defences | 40 min |
| 05 | [Percolator deep dive](05-percolator-deep-dive.md) | Upstream vs the dcccrypto fork, and what you can build on them | 60 min |
| 06 | [Established venues](06-established-venues.md) | Who runs what design, and how they are doing | 40 min |
| 07 | [Open-source code to read](07-open-source-code.md) | Repositories, licences, and what each teaches | reference |
| 08 | [Solana infrastructure for perps](08-solana-infra.md) | Oracles, block building, what is live vs proposed | 30 min |
| 09 | [Problems and incidents](09-problems-and-incidents.md) | Exploits and failures, each with its design lesson | 45 min |
| 10 | [Gaps and what to build](10-gaps-and-what-to-build.md) | Open problems ranked, including long-tail token perps | 45 min |
| 11 | [Reading list](11-reading-list.md) | Papers, docs and posts, in the order to consume them | weeks |

## Suggested four-week plan

**Week 1 — mechanics.** Files 01–04. Then trade a small position on two venues of different designs (one order book, one pool) on testnet or with trivial size, and write down every number the UI shows you and how it is computed.

**Week 2 — existing systems.** Files 06, 07, 09. Pick one order-book codebase and one pool codebase from file 07 and trace a single trade and a single liquidation through each.

**Week 3 — Percolator.** File 05, then the reading order at the bottom of it. Run upstream's test suite. Write a toy matcher.

**Week 4 — your thesis.** Files 08 and 10. Write one page answering the five questions at the end of file 03 for the product you want to build.

## Three things to hold onto

1. The matching design is the visible choice; the risk engine is the one that decides survival.
2. On a one-sided market somebody has to be the counterparty. Every long-tail perp design is an answer to "who, and why would they?".
3. Every public codebase you will read for Percolator says it is unaudited. Read them to learn, and verify before you deploy anything with funds.
