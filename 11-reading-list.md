# 11 — Reading list

In the order to consume them. Unless noted, each URL returned a page when checked on 2026-11-12. Article titles outside arXiv are as I recall them and may differ slightly from the page.

## Stage 1 — How perps work (2–3 days)

1. **The Cartoon Guide to Perps** — Paradigm. Start here. https://www.paradigm.xyz/writing/the-cartoon-guide-to-perps
2. **A primer on perpetuals** — arXiv 2209.03307. https://arxiv.org/abs/2209.03307
3. **Hyperliquid docs: funding** https://hyperliquid.gitbook.io/hyperliquid-docs/trading/funding
4. **dYdX docs: funding** https://docs.dydx.xyz/concepts/trading/funding
5. **BitMEX perpetual contracts guide** — the original design. The site blocks automated checks, so this link was not verified. https://www.bitmex.com/app/perpetualContractsGuide

Pair with files 01 and 02.

## Stage 2 — How venues are built (1 week)

Read one venue's documentation end to end before sampling others. Hyperliquid's is the most complete.

6. **Hyperliquid: liquidations** https://hyperliquid.gitbook.io/hyperliquid-docs/trading/liquidations
7. **Hyperliquid: auto-deleveraging** https://hyperliquid.gitbook.io/hyperliquid-docs/trading/auto-deleveraging
8. **Hyperliquid: robust price indices** https://hyperliquid.gitbook.io/hyperliquid-docs/trading/robust-price-indices
9. **Hyperliquid: oracle** https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/oracle
10. **Hyperliquid: protocol vaults (HLP)** https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/vaults/protocol-vaults
11. **Hyperliquid: hyperps** — perps with no spot oracle. https://hyperliquid.gitbook.io/hyperliquid-docs/trading/hyperps
12. **Hyperliquid: HIP-3 builder-deployed perps** https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-3-builder-deployed-perpetuals
13. **Phoenix perps overview** https://docs.phoenix.trade/phoenix/perpetual-futures.md
14. **Phoenix matching engine** (splines) https://docs.phoenix.trade/phoenix/matching-engine/matching-engine.md
15. **Phoenix mark price** https://docs.phoenix.trade/phoenix/margin-and-risk/mark-price.md
16. **Jupiter Perps technical reference** https://docs.jup.ag/user-docs/trade/perps/technical-reference
17. **Velocity (ex-Drift) protocol docs** https://docs.velocity.exchange/protocol
18. **Velocity design notes** in the repo https://github.com/velocity-exchange/velocity-v1/tree/master/docs
19. **Chaos Labs: GMX risk framework methodology** — how a pool venue's parameters are set. https://chaoslabs.xyz/posts/chaos-labs-gmx-genesis-risk-framework-methodology

Pair with files 03, 06 and 07.

## Stage 3 — How venues fail (1 week)

### Oracle manipulation

20. **CertiK: Mango Market incident analysis** https://www.certik.com/blog/mango-market
21. **Solidus Labs: order-book analysis of the Mango attack** https://www.soliduslabs.com/post/mango-hack
22. **Chainlink: market manipulation vs oracle exploits** https://chain.link/education-hub/market-manipulation-vs-oracle-exploits
23. **Chainalysis: oracle manipulation attacks** https://www.chainalysis.com/blog/oracle-manipulation-attacks-rising/
24. **Halborn: KiloEx hack** https://www.halborn.com/blog/post/explained-the-kiloex-hack-april-2025
25. **Halborn: Hyperliquid POPCAT** https://halborn.com/blog/post/explained-the-hyperliquid-hack-november-2025
26. **BlockSec: deep dive into HIP-3** https://blocksec.com/blog/deep-dive-into-hip-3-a-builder-centric-perspective
27. **Nexus Mutual: SKHYNIX flash crash report** https://nexusmutual.io/blog/xyz-skhynix-flash-crash-on-hyperliquid-incident-report

### Governance and operations

28. **BlockSec: Drift incident — multisig compromise via durable nonces** https://blocksec.com/blog/drift-protocol-incident-multisig-governance-compromise-via-durable-nonce-exploitation
29. **Range: Inside the Drift exploit** https://range.org/blog/inside-the-drift-exploit-how-to-protect-your-protocol-and-multisigs-against-dprk-threat-actors
30. **Hypernative: when privileged access has no limits** https://www.hypernative.io/insights/blog/the-drift-exploit-when-privileged-access-has-no-limits
31. **Drift → Velocity migration notes** https://github.com/velocity-exchange/velocity-v1/blob/master/docs/DRIFT-TO-VELOCITY.md

### Liquidation and ADL

32. **Inside the $19B flash crash** https://insights4vc.substack.com/p/inside-the-19b-flash-crash
33. **Autodeleveraging: Impossibilities and Optimization** — Tarun Chitra, arXiv 2512.01112. Read v3; the headline figure changed between versions. https://arxiv.org/abs/2512.01112
34. **Autodeleveraging, Hyperliquid, and the $653m debate** — a readable walk-through of the paper and the pushback. https://thogiti.github.io/2025/12/11/Autodeleveraging-Hyperliquid-653M-debate.html
35. **Autodeleveraging as Online Learning** — arXiv 2602.15182. https://arxiv.org/abs/2602.15182
36. **Blockworks: Hyperliquid and the "validator put"** (JELLY) https://blockworks.co/news/hyperliquid-decentralization-validator-put

Pair with files 04 and 09.

## Stage 4 — Percolator (1–2 weeks)

37. **dcccrypto/percolator README** — the plain-language explanation of `h`, `A/K/F` and the envelope. https://github.com/dcccrypto/percolator
38. **Percolator spec, section 0** https://github.com/aeyakovenko/percolator/blob/master/spec.md
39. **percolator-prog README** — product specification and trust boundaries. https://github.com/aeyakovenko/percolator-prog
40. **percolator-match** https://github.com/aeyakovenko/percolator-match
41. **Permissionless perps for long-tail tokens — Design A** https://github.com/dcccrypto/percolator-perp-liquidity
42. **0xArchive: the HIP-3 deployer bottleneck** — why capital-gated permissionless listing did not reach the long tail. https://0xarchive.io/resources/research/addressing-hyperliquid-s-hip-3-deployer-bottleneck
43. **Kani documentation** https://model-checking.github.io/kani/

Pair with files 05 and 10.

## Stage 5 — Solana market structure (2–3 days)

44. **Anza: The Internet Capital Markets Roadmap** https://www.anza.xyz/blog/the-internet-capital-markets-roadmap
45. **Helius: Solana's proprietary AMM revolution** https://www.helius.dev/blog/solanas-proprietary-amm-revolution
46. **Helius: Block Assembly Marketplace** https://www.helius.dev/blog/block-assembly-marketplace-bam
47. **Helius: Constellation** (multiple concurrent proposers) https://www.helius.dev/blog/constellation
48. **Anza: Alpenglow** https://www.anza.xyz/blog/alpenglow-a-new-consensus-for-solana
49. **Pyth Core upgrade notes** https://docs.pyth.network/price-feeds/core/upgrade

Pair with file 08.

## Stage 6 — Theory, when you want depth

50. **Fundamentals of Perpetual Futures** — arXiv 2212.06888. https://arxiv.org/abs/2212.06888
51. **Perpetual Futures Pricing** — arXiv 2310.11771. https://arxiv.org/abs/2310.11771
52. **Designing funding rates for perpetual futures** — arXiv 2506.08573. https://arxiv.org/abs/2506.08573
53. **Everlasting Options** — Paradigm. https://www.paradigm.xyz/writing/everlasting-options
54. **Power Perpetuals** — Paradigm. https://www.paradigm.xyz/writing/power-perpetuals
55. **Floor Perps** — Paradigm. https://www.paradigm.xyz/writing/floor-perps

## Live data to keep open

- Open interest by venue: https://api.llama.fi/overview/open-interest
- Volume by venue: https://api.coingecko.com/api/v3/derivatives/exchanges
- DefiLlama perps dashboard: https://defillama.com/perps

## Links that are dead

Older reading lists point at these; they no longer resolve.

- `osec.io/blog/2022-10-12-mango-markets-exploit`
- `blog.neodyme.io/posts/mango_markets_exploit/`
- `paradigm.xyz/2023/03/perps-funding`
- `umbraresearch.xyz/writings/toxic-flow`
- Deep links under `docs.drift.trade` (now redirects to Velocity; old paths error)
- Anything on Switchboard

## How to read

- For each paper or post, write three lines: the mechanism, the failure it addresses, and whether it applies to a one-sided long-tail market.
- For each incident, write the attacker's cost and payoff. If you cannot, you have not understood it yet.
- Trust venue docs for what a venue intends and incident reports for what it does under stress.
