# 09 — Problems and incidents

Each incident is here for the design lesson. Dollar figures come mostly from press and security-firm write-ups and differ between sources; treat them as approximate. Researched 2026-11-12.

## The pattern behind most of them

Almost every loss below is one of four things:

1. **Paper profit treated as money** — manipulated unrealised PnL was withdrawn or borrowed against.
2. **Someone could write the price** — an oracle key, relayer or setter function was reachable.
3. **A shared pool inherited risk it did not price** — a backstop vault or LP pool took the other side of an attack.
4. **An admin key could do too much** — listings, limits or custody changed by a small signer set.

When you design, ask of every feature which of these four it could open.

## Incidents

### Mango Markets — 11 Oct 2022, ~$110–116M

- **What happened:** A trader took large MNGO-perp positions across two accounts, pumped thin MNGO spot on the venues feeding the oracle (about 13× in 30 minutes), then borrowed the treasury against the winning account's paper profit.
- **Root cause:** The oracle was accurate. Unrealised profit on a manipulable asset counted as collateral, and OI was not capped relative to spot depth.
- **Lesson:** Unrealised PnL must not be withdrawable collateral without a rule for whether it is backed. Cap OI against real spot liquidity.
- Source: https://www.cftc.gov/PressRoom/PressReleases/8647-23

### GMX AVAX — 18 Sep 2022, ~$565K

- **What happened:** Zero-slippage oracle execution let a trader open size on GMX, move the reference CEX price, close, repeat.
- **Lesson:** Oracle-priced fills need price impact and OI caps tied to external depth.
- Source: https://cointelegraph.com/news/decentralized-exchange-gmx-suffers-565k-price-manipulation-exploit

### Hyperliquid JELLY — 26 Mar 2025

- **What happened:** A trader opened a multi-million short, pulled margin so the protocol vault (HLP) inherited the position, then pumped spot. HLP's unrealised loss reached about $13.5M. Validators delisted the market and settled at the attacker's entry price.
- **Lesson:** A shared backstop vault that inherits positions is an attack surface. A discretionary rescue fixes the loss and damages credibility.
- Source: https://blockworks.co/news/hyperliquid-decentralization-validator-put

### KiloEx — 14 Apr 2025, ~$7–7.5M

- **What happened:** A forwarder contract lacked access control. The attacker called the price setter, set ETH to $100, opened longs, set it to $10,000.
- **Lesson:** Price-write authority is the most valuable permission in the system. Bound it on-chain by deviation and freshness, whoever holds the key.
- Source: https://www.halborn.com/blog/post/explained-the-kiloex-hack-april-2025

### GMX V1 — 9 Jul 2025, ~$42M

- **What happened:** Cross-contract reentrancy distorted the global short average price, inflating the LP token's redemption value. The bug was reportedly introduced by an earlier fix.
- **Lesson:** LP share price derived from mutable global PnL accounting is fragile. Test conservation invariants, especially after patches.
- Source: https://blocksec.com/blog/gmx-incident-cross-contract-reentrancy-bypasses-a-four-year-old-guard

### The 10–11 Oct 2025 cascade — ~$19B liquidated market-wide

- **What happened:** Hyperliquid triggered cross-margin ADL for the first time, closing about $2.1B of positions in 12 minutes. Other venues ran ADL too.
- **Analysis:** Tarun Chitra's paper estimates Hyperliquid's queue over-haircut winners by $45–52M and that Binance overused ADL by more. It argues no policy satisfies solvency, revenue and fairness together.
- **Lesson:** ADL policy is a product feature, not a footnote. Pro-rata allocation beats ranked queues on fairness.
- Source: https://arxiv.org/abs/2512.01112v3

### Hyperliquid POPCAT — 12 Nov 2025, ~$4.9M vault bad debt

- **What happened:** About $3M across 19 wallets built roughly $20M of longs behind a buy wall, then pulled the wall. The attacker burned their own margin to hand the vault a loss.
- **Repeats:** FARTCOIN (9 Apr 2026, ~$1.5M to HLP) and XPL (1 Nov 2026, ~$556K).
- **Lesson:** An attacker can lose on-venue while hedged elsewhere. Per-asset OI and leverage must scale with real exit liquidity, and per-account limits are defeated by many wallets.
- Sources: https://halborn.com/blog/post/explained-the-hyperliquid-hack-november-2025 · https://cointelegraph.com/news/trader-loses-3m-fartcoin-hyperliquid-adl-liquidation

### HIP-3 oracle incidents — Dec 2025 and Jul 2026

- **XYZ100 (14 Dec 2025):** a ~$10M short moved the market over 3% in a minute, liquidating ~$3M. The oracle came from the deployer's own relayer.
- **SKHYNIX (28 Jul 2026):** the oracle took an 18% Korean pre-market print as reference and triggered a cascade. The deployer said the oracle "worked as designed" and reimbursed at its discretion. Slashed stake is burned, so victims get nothing from slashing.
- **Lesson:** Permissionless market deployment moves oracle risk to whoever deploys. Off-hours pricing for assets with trading sessions is a design problem in its own right.
- Sources: https://blocksec.com/blog/deep-dive-into-hip-3-a-builder-centric-perspective · https://nexusmutual.io/blog/xyz-skhynix-flash-crash-on-hyperliquid-incident-report

### Drift — 1 Apr 2026, ~$285–295M

The most important one for a Solana builder.

- **What happened:** Not a contract bug. Signers were socially engineered into pre-signing durable-nonce transactions. The security council had moved to 2-of-5 with no timelock days earlier. The attacker listed a fake token with a few thousand dollars of pool liquidity as collateral, raised withdrawal limits, and drained in 31 withdrawals over about 12 minutes. Attributed to North Korean actors.
- **Aftermath:** Drift relaunched as Velocity, perps-only. About $155–159M of Jupiter's JLP token sat in the drained vaults — contagion through composability, not a Jupiter bug.
- **Lesson:** Admin power over listings and limits was the largest risk on Solana's largest perp venue. Use timelocks, minimum-liquidity listing gates, withdrawal rate limits, or no admin keys at all.
- Sources: https://www.trmlabs.com/resources/blog/north-korean-hackers-attack-drift-protocol-in-285-million-heist · https://phemex.com/news/article/jupiter-confirms-jlp-fully-backed-amid-drift-incident-70369

### Ostium — 15 Jul 2026, ~$12–18M

- **What happened:** One batched transaction delivered fabricated BTC prices ($5,000) and traded against them. Root cause was not established in the analysis reviewed; a compromised signer key is one candidate. Audits had excluded key custody and price-upkeep contracts.
- **Lesson:** KiloEx again, 15 months later. Audit scope must include the price path.
- Source: https://defiprime.com/ostium-exploit

### Custody and transfer bugs — 2026

- **AFX Trade (Arbitrum, Jul 2026, ~$24M)** through its own USDC custody bridge.
- **Arcus (Nov 2026, ~$2.2M)** through a sub-account transfer bug.
- **Lesson:** Bridge custody and internal-transfer accounting are recurring loss points, unrelated to perp logic.
- Sources: https://decrypt.co/374133/arbitrum-perp-dex-afx-trade-drained-of-24m-offers-hacker-30-to-return-it · https://cryptoticker.io/en/arcus-exploit-perp-dex-sub-account-bug/

### Not confirmed

- **Oracle latency exploitation** is well described as a mechanism, but no verified 2025–26 loss figure was found.
- **Jupiter Perps** — no direct exploit or depeg found.

## Structural problems, 2026

| Problem | State of play | Source |
|---|---|---|
| **ADL fairness** | Unresolved since Oct 2025; venues now compete on ADL parameters and insurance size | https://cryptobriefing.com/crypto-accountability-market-crash-debate/ |
| **Position privacy** | Public liquidation prices get hunted. Private perps exist via NEAR-over-Hyperliquid and ZK books; no Solana-native venue with confirmed volume found | https://blog.synthetix.io/private-perps-the-end-of-target-practice/ |
| **Portfolio margin** | Hyperliquid's is in beta (Jun 2026). No Solana equivalent found | https://x.com/HyperliquidX/status/2069992300114296839 |
| **Funding-rate fragmentation** | Pendle Boros trades funding rates (~$200M OI) and launched cross-venue spread markets in Oct 2026. Nothing equivalent found on Solana | https://cryptobriefing.com/boros-launches-funding-rate-spread-markets/ |
| **Basis yield compression** | Crypto basis yields fell; Ethena moved into equity-perp basis | https://cryptoslate.com/ethena-is-targeting-the-120-trillion-wall-street-stocks-market-to-hunt-yields-5x-higher-than-bitcoin/ |
| **Margin fragmentation** | 20+ perp DEXs, no shared margin. Solana's main aggregator (Ranger) shut down in May 2026 | https://solanafloor.com/news/ranger-finance-winds-down |
| **Points farming** | Volume roughly halves after token launch; Lighter's fell sharply post-airdrop | https://en.theblockbeats.news/news/60245 |
| **Deployer concentration** | One deployer holds over 95% of HIP-3 volume | https://0xarchive.io/resources/research/addressing-hyperliquid-s-hip-3-deployer-bottleneck |
| **Oracles for unlisted assets** | Deployer-run relayers, off-hours equity pricing and DEX-pool averages are the weak point of every long-tail design | https://blocksec.com/blog/deep-dive-into-hip-3-a-builder-centric-perspective |
| **Shrinking market** | Total perp OI fell from ~$210B (pre-Oct 2025) to ~$99B (Apr 2026) | https://beincrypto.com/crypto-leverage-perp-trading-report-coingecko/ |

### Regulation

- **US:** The CFTC approved a first BTC perp in May 2026; Kraken and Coinbase launched US perps in June 2026; single-stock perps were cleared on 3 Nov 2026 and are already being challenged in court. Onshore competition for majors and equities is now real. https://www.coindesk.com/policy/2026/11/03/cftc-clears-perpetual-futures-on-stocks-bringing-crypto-trading-model-to-wall-street
- **EU:** ESMA's Feb 2026 statement treats crypto perps as likely CFDs, which carries retail leverage limits. https://www.financemagnates.com/forex/regulation/esma-tells-firms-perpetual-futures-fall-under-eu-cfd-rules/

This is not legal advice; if you intend to serve users in either region, get a lawyer before you write the front-end.

## Solana venue attrition in 2026

Worth absorbing before you pick a design:

- **Drift** — exploited, relaunched as Velocity.
- **Flash Trade** — wound down, announced Aug 2026, citing team, market and economics. https://solanacompass.com/news/flash-trade-winds-down-solana-perpetuals-exchange-after-three-years-seeks-acquirer
- **Ranger** — treasury liquidated by token-holder vote, then shut down.
- **pump.fun perps** — launched Oct 2026 but routes to Hyperliquid, with no perps on its own launched tokens. https://www.cryptotimes.io/2026/10/28/pump-fun-launches-perps-for-crypto-stocks-commodities-with-hyperliquid-integration/
- Solana was cited at about 1% of global perp volume. https://solanacompass.com/news/lily-liu-defends-solana-foundation-against-perps-kingmaking-claims-as-flash-trade-exits

## Percolator: what could and could not be confirmed

- **Confirmed from the repos:** both engine and wrapper are labelled unaudited and educational. The proofs cover engine invariants, not a whole exchange.
- **Reported by press:** the repo appeared in Oct 2025 as a sharded order-book design; in Feb 2026 the author called it "basically feature complete" as a risk library where users bring their own matchers; around Apr 2026 he deployed an immutable binary with 5 SOL of insurance and burned admin keys as an open challenge. Current status of that deployment was not confirmed. https://solanafloor.com/news/return-open-source-immutable-software-solana-cofounder-issues-challenge-defi-community
- **Tokens:** pump.fun tokens using the name have traded. No evidence of an official token was found, and no explicit disavowal either.
- **Name-riding:** at least one GitHub organisation copies the README, shows the author's handle as a badge and advertises a token. Treat any "Percolator" product as unaffiliated until proven otherwise.
- **Third-party review of Percolator Launch (Apr 2026):** scored it a fail, citing a critical stake-pool bug, a matcher upgradeable by one wallet, and an associated token with a doubtful deployer history. The core program was found immutable. The review is seven months old and the code has changed since. https://trenchguard.xyz/report/percolator-launch-tg-2026-0002/print
