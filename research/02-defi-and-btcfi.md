# 02 — Cardano DeFi & Bitcoin DeFi (BTCfi): what exists, what you can build on, where the gaps are

*Research snapshot: 2026-10-07. Audience: engineers coming from Ethereum, Solana or web2. Live numbers were pulled from DefiLlama's public API on 2026-10-07 (`api.llama.fi`, `stablecoins.llama.fi`) and governance data from Koios (`api.koios.rest/api/v1/proposal_list`). Anything I could not confirm is marked **[UNVERIFIED]**.*

---

## TL;DR for builders

- **Cardano DeFi is small and has been shrinking.** DefiLlama puts chain TVL at **$72.7M** (rank **#36** of all chains) on 2026-10-07. That is down from $169.8M on 2026-01-01 and from an all-time high of $721M on 2024-12-07. ADA fell from about $0.33 to $0.27 over the same period (it touched about $0.17 in August), which explains much of the drop. 24h DEX volume is about **$4.0M** (rank about **#40**).
- **2026 was the "infrastructure catch-up" year.** Treasury money (the ₳70M Critical Integrations budget) brought **USDCx** (Circle xReserve-backed USDC, live 2026-02-27), **Pyth Pro** pull oracles, **Dune**, and a phased **LayerZero** integration. USDC (mostly USDCx) is now about **$46.6M**, roughly 69% of Cardano's $67.4M stablecoin supply.
- **Cardano PRIME** is a ₳120M (about $19.2M) treasury-funded, 12-month DeFi growth program run by **AlphaGrowth**. It was enacted in August 2026 with 73% DRep support. Its Phase-1 audit gave Cardano DeFi a score of **51.3/100** (Ethereum 96.3, Sui 72.3), and it has opened an RFP. The RFP's named priorities are: **dual-stack (EVM+Cardano) wallets, an EVM→Cardano bridge, lending & vault standards, and liquidity tooling**. The rule of thumb is **$50 of sustained TVL per $1 of grant**.
- **Bitcoin DeFi on Cardano is still mostly pre-mainnet.** Cardinal (IO + BitVMX) demoed a mainnet transfer in May 2025. Since then, three treasury requests for BTCfi failed in 2026: IO's **Pogun**, FluidTokens' **Bifrost**, and Sundial×Charms **Alchemy**. Hoskinson says Pogun ships around mid-December 2026 anyway. Bifrost is on BTC testnet. The only BTC exposure live today is synthetic (Indigo **iBTC**) or comes from small federated/MPC bridges (Rosen, Wanchain).
- **Many tools are open source and you can compose with them.** Minswap SDK, SundaeSwap SDK, Indigo SDK plus the `dexter` aggregator lib, Splash core, WingRiders v2 contracts, Strike's perps contracts, the DexHunter partner API, Charli3 pull-oracle client, Pyth Lazer Cardano (Aiken), Orcfax feeds, the DeFi Kernel registry, and the FluidTokens Bifrost bridge repo.

---

## 1. DeFi landscape (October 2026)

### 1.1 Headline numbers (DefiLlama, pulled 2026-10-07)

| Metric | Value | Notes |
|---|---|---|
| Chain DeFi TVL | **$72.7M** | Rank #36. Just behind Mezo/Mixin, ahead of PulseChain/dYdX ([DefiLlama chains](https://defillama.com/chains)) |
| TVL all-time high | $721M (2024-12-07) | Same API (`/v2/historicalChainTvl/Cardano`) |
| TVL trajectory 2026 | $169.8M (Jan 1) → $140M (Mar 5, post-USDCx) → $129M (Jun 1) → $73.8M (Jul 1) → $58M (Sep 1) → $72.7M (Oct 7) | The sharp late-June drop coincides with a broad DeFi contraction (DeFi TVL −39% YTD, [Cointelegraph](https://cointelegraph.com/news/defi-tvl-falls-39-2026-erases-45b-value)) and the July SecondFi wallet exploit (below). The exact attribution of the June step-down is **[UNVERIFIED]** |
| ADA price | $0.333 (Jan 1) → $0.235 (Jun 1) → $0.168 (Aug 1) → **$0.267** (Oct 7) | [DefiLlama coins API](https://coins.llama.fi/prices/current/coingecko:cardano) |
| Stablecoin supply on Cardano | **$67.4M** | [DefiLlama stablecoins](https://defillama.com/stablecoins/Cardano) |
| DEX volume | 24h $4.0M; 7d $42.7M; 30d $82.3M | Rank about #40 by 24h DEX volume (Solana #1 at $2.4B/day) |

For context: Ethereum is about $54B TVL, Solana $6.6B, Base $6.4B, Bitcoin (BTCfi L1 accounting) $4.6B. Cardano's TVL is roughly the size of a single mid-tier protocol on Solana.

### 1.2 Stablecoins on Cardano (DefiLlama, 2026-10-07)

| Stablecoin | Supply on Cardano | Type / issuer | Notes |
|---|---|---|---|
| **USDC / USDCx** | **$46.6M** | Fiat, Circle-backed via **xReserve** | USDCx launched 2026-02-27. You deposit USDC into Circle's xReserve contract on Ethereum and USDCx is minted 1:1 on Cardano; burning it releases USDC. Issued by a protocol on Cardano with Circle attestations, *not* native Circle issuance in the CCTP sense ([Circle](https://www.circle.com/blog/usdcx-on-cardano-now-available-via-circle-xreserve), [Intersect](https://intersectmbo.org/news/critical-integrations-technical-progress-update-and-pentad-v2)). Mainnet asset `asset1e7eewpjw8ua3f2gpfx7y34ww9vjl63hayn80kl`, testnet `asset1ejelsh8crza8dyghxzsjhkjqutzr7q3dnregng`. Bridge UI: [usdcx.iog.io/bridge](https://usdcx.iog.io/bridge). DefiLlama's "USDC" figure probably also includes a small amount of Wanchain-wrapped USDC (about $0.4M historically) |
| **USDM** (Moneta) | $13.6M | Fiat-backed, Cardano-native | Bank deposits plus MMFs ([USDM blog](https://medium.com/@USDMOfficial/the-shift-to-independent-stability-why-usdm-is-the-new-b2b-treasury-standard-on-cardano-eddb7f2a5fd5)) |
| **USDA** (Anzens) | $4.2M | Fiat-backed | Received ₳4M from the 2025 Intersect budget for wallet/exchange/ramp support (Koios gov action, July 2025) |
| **DJED** | $1.44M supply (protocol TVL $6.7M) | Over-collateralized algorithmic (IOG design, COTI-issued) | Reserve ratio 400–800% design |
| **iUSD** (Indigo) | $1.40M | CDP synthetic | — |
| USDT | $0.11M | Wrapped (Wanchain) | **There is no native Tether on Cardano** |
| **USDr** (RealFi) | n/a yet | RWA-backed (T-bills, MMFs, private credit); sUSDr yield | RealFi went live on mainnet **2026-10-01** with Lace, Liqwid and SundaeSwap integrations ([KuCoin](https://www.kucoin.com/news/flash/realfi-launches-cardano-native-stablecoin-backed-by-real-world-assets), [The Crypto Basic](https://thecryptobasic.com/2026/10/05/what-the-hell-else-can-i-do-cardano-founder-defends-ada-progress)) |

### 1.3 Protocols by category (DefiLlama Cardano TVL, 2026-10-07)

| Category | Protocol | Cardano TVL | 30d volume | Notes |
|---|---|---|---|---|
| DEX (AMM) | **Minswap** | $18.9M (Jan: $46.5M) | $15.4M | Largest DEX. V2 contracts; stableswap pools; aggregator widget (Feb 2026). Open-source SDK |
| DEX (AMM) | **SundaeSwap** V2/V3/V4 | $0.67M / $2.8M / $0.43M | V2: $46.0M | V3 TVL fell from $6.5M to $2.8M at end of June 2026. V2 shows the highest 30d volume of any Cardano DEX on DefiLlama |
| DEX (bond/order book) | **Dano Finance** | $7.1M | $14.8M | Trades Optim bond tokens. Champions the **DeFi Kernel** open order-book standard |
| DEX (hybrid order book + AMM) | **Splash** | $2.73M | $0.17M | Off-chain execution engine anyone can run. Autonomous accounts / MM strategies ([cardano.org](https://cardano.org/apps/splash/)) |
| DEX | **WingRiders** | $1.53M | $5.8M | V2 contracts public |
| DEX | VyFinance $0.75M; CSWAP $0.68M; MuesliSwap $0.20M; TeddySwap $0.11M; Genius Yield about $0.01M | | | Genius Yield (order book) is nearly empty |
| DEX (Hydra L2 order book) | **DeltaDeFi** | about $0 | — | Launched in mainnet beta in 2025. **Paused operations on 2026-07-15 for lack of funding** ([DefiLlama](https://defillama.com/protocol/deltadefi), [search summary](https://cryptonews.net/news/defi/33159615/)) |
| Lending (pooled) | **Liqwid** | $13.8M (Jan: $42.4M) | — | Largest money market. About 3M USDCx supplied in the first week after launch ([IOG](https://www.iog.io/news/usdcx-on-cardano-here-s-what-happened-in-the-first-seven-days)) |
| Lending (pooled, isolated markets) | **Surf Lending** | $4.0M | — | Isolated markets ([cardano.org](https://cardano.org/apps/surf-lending/)) |
| Lending (P2P / BTCfi infra) | **FluidTokens** | $3.7M (Jan: $1.3M, one of the few that grew) | — | P2P loans v4, Aquarium (babel-fee "fee tanks"), Bifrost BTC bridge |
| Lending (other) | Lenfi $0.18M; Yamfore $0.07M; Levvy; CherryLend | | | Long tail |
| CDP / synthetics | **Indigo** | $4.8M | — | iUSD, iBTC, iETH, iSOL synthetics, ADA-collateralized. Open SDK |
| Perps / derivatives | **Strike Finance** | $1.94M | — | V2 launched 2026-03-20: $87.3M volume and $30.1K revenue through 2026-06-15. Strike claims >$1.13B cumulative volume and ">50% of Cardano trading activity" ([Strike treasury proposal metadata via Koios](https://www.cardanocube.com/governance/gov_actions/gov_action1suskjc6c4nw58c6wtmv77xe79gwj47wp4gvh9cqhhxujwxmam3cqqkz5nwj?tally=live)). Its 9M ADA treasury liquidity request **expired** (June 2026) |
| Staking rental / bonds | **Optim Finance** | $0.77M | — | Bond tokens (traded on Dano) |
| Staking baskets | **Atrium** | $0.53M | — | Delegation "baskets" across 50 SPOs |
| Prediction markets | Bodega | about $0 | — | Effectively empty category |
| Aggregators | **DexHunter**, **Steelswap**, Minswap aggregator, Indigo `dexter` lib | n/a | — | DexHunter routes across 15+ DEXes; partner API with rev-share ([docs](https://dexhunter.gitbook.io/dexhunter-partners/api-reference/api)) |
| Bridges (counted on Cardano) | Wan Bridge $0.76M; Rosen $0.32M; ChainPort $0.28M; NEAR Intents $0.15M | | | NEAR Intents lists Cardano as a supported chain |

Notes:
- DefiLlama also attributes **$15.5M "Coinbase Bridge"** to Cardano. As far as I can tell, this is ADA held by Coinbase to back its wrapped ADA on other chains, not DeFi on Cardano. It is excluded from the chain TVL figure **[interpretation, UNVERIFIED]**.
- Year-to-date drops (January → October 2026) are large across the board: Minswap −59%, Liqwid −68%, Indigo −51%, Splash −64%. Only FluidTokens and Surf held steady or grew.

### 1.4 Oracles

| Oracle | Model | Status | Builder notes |
|---|---|---|---|
| **Pyth Pro (Lazer)** | Pull. Signed updates are fetched off-chain and verified on-chain with a **withdraw-zero** pattern (withdraw 0 lovelace from the Pyth withdraw script, updates in the redeemer, Pyth state as a reference input). The consuming contract must enforce freshness | Live via Critical Integrations. Developer portal lists it as beta/"maintainer pick". Preprod examples exist | `@pythnetwork/pyth-lazer-sdk` (TS) plus the Aiken lib [`pyth-network/pyth-lazer-cardano`](https://github.com/pyth-network/pyth-lazer-cardano). Needs an access token (`LAZER_TOKEN`). More than 125 institutional publishers ([Pyth docs](https://docs.pyth.network/price-feeds/pro/integrate-as-consumer/cardano), [dev portal](https://developers.cardano.org/tools/pyth-pro/)) |
| **Charli3** | Push feeds plus **On-Demand Validation (ODV)** pull oracle | Mainnet plus preprod | [`charli3-pull-oracle-client`](https://github.com/Charli3-Official/charli3-pull-oracle-client), [docs](https://docs.charli3.io/oracles/products/pull-oracle/summary). Partially open-sourced (node software) |
| **Orcfax** | Heartbeat / on-demand publications. CER feeds plus CNT feeds aggregated from DEX pools | Mainnet | [`orcfax/cer-feeds`](https://github.com/orcfax/cer-feeds), [dev portal](https://developers.cardano.org/docs/build/integrate/oracles/orcfax) |

The PRIME audit explicitly calls for a **"second independent price-feed provider"** ([DailyCoin](https://dailycoin.com/cardanos-200m-defi-push-puts-evm-liquidity-in-focus/)). Oracle redundancy is still treated as a gap.

### 1.5 Liquid staking & yield
- ADA staking is natively liquid: no lock-up, no slashing, and delegated ADA stays spendable. So "LSTs" matter less than on Ethereum. The real issue is that **ADA locked in DeFi contracts often loses its staking rewards** unless the protocol attaches a stake credential. The OpenZeppelin proposal (below) still lists a **Liquid Staking Protocol** reference implementation as a strategic gap.
- Yield sources: Liqwid/Surf supply APY, DEX LP fees, Optim bonds, Atrium baskets, USDr/sUSDr (RealFi). There is no sizable restaking, structured-vault, or delta-neutral stablecoin (Ethena-style) market.

---

## 2. Treasury-funded DeFi & liquidity initiatives (2025–2026)

Cardano's on-chain treasury is the dominant funding source for ecosystem-level DeFi infrastructure. All actions below come from Koios (`proposal_list`) unless otherwise cited.

### 2.1 Cardano PRIME (AlphaGrowth), the big one

| Item | Detail |
|---|---|
| Name | **PRIME = Protocol Readiness, Incentives, and Market Expansion** ([alphagrowth.io/cardano-prime](https://alphagrowth.io/cardano-prime)) |
| Operator | **AlphaGrowth** (blockchain growth/research firm). Leads on the X Space were Eric Waisanen and Bryan Colligan ([cardano-note PR](https://github.com/shiodome47/cardano-note/pull/3)) |
| Governance action | `gov_action122wue2k65qq8gmpz795z2axt8apka6ay6xt3pwg8jxj5yfkujmtsqvlfpu7`. Submitted 2026-07-09/10 (epoch 642), expiry epoch 649 ([danonight](https://danonight.com/news/cardano-prime-treasury-withdrawal)). Status: **enacted** (Koios) |
| Vote | 73.04% yes (~3.73B ADA) vs 26.96% no; ~5.11B of 15.01B ADA participated ([The Crypto Basic, 2026-08-12](https://thecryptobasic.com/2026/08/12/cardano-community-approves-120-million-ada-allocation-to-boost-defi-liquidity/)). Cardano Foundation voted yes ([CF on X](https://x.com/Cardano_CF/status/2087164970353979629)) |
| Amount | **₳120,000,000** (about $19.2M at a $0.16 planning rate). About ₳30M for Phases 1–2; about **₳90M gated** behind a Month-4 Operating Group vote |
| Target | +$200M net "qualifying" TVL in 12 months (about $90M → $290M). That works out to about $0.073 of spend per $1 of TVL. A falsification trigger applies if growth is below $80M by Month 6 |
| Phases | **P1 (M1–2):** current-state audit across 20–25 categories, published as a public good. **P2 (M2–4):** gap analysis and the "Integration and Ecosystem Support Recommendations" document. **P3 (M5–12):** milestone/action-gated incentives, LP dealmaking, cohort liquidity campaigns, seed capital, structured yield products |
| Budget | AG fixed fee $1.76M (₳11M). **Ecosystem Grants $5.6M (₳35M): $3M in Phase 2, $3M gated in Phase 3**. LP incentives $4.32M (₳27M, all Phase 3). Marketing $2.4M (revised down to about $648,720 in the final version, per [The Crypto Basic](https://thecryptobasic.com/2026/08/12/cardano-community-approves-120-million-ada-allocation-to-boost-defi-liquidity/)). Legal $0.16M. Independent audit $0.32M. Performance-fee reserve $4.64M (paid only against verified TVL growth: 3%/2%/1% bands, capped at $264M of growth) |
| Grant limits | No single grant over **$1M** or over 25% of the Ecosystem Grants envelope (proposal "Exclusions") |
| Oversight | **Operating Group** of 5 unpaid members: Christina Gianelloni (Blink Labs), Kavinda Kariyapperuma (CoinCeylon), Philip DiSarro (Midgard Labs), Gerard Moroney (IOG), Kristijan Kowalsky (Tweag/Modus Create). 3-of-5 veto. **Intersect** is the Constitutional Administrator and holds the funds (institutional custody, BitGo considered). A non-voting Advisory Council of DEX, lending, stablecoin, perps, LST and yield teams supports the OG ([danonight](https://danonight.com/news/cardano-prime-treasury-withdrawal), [AlphaGrowth](https://alphagrowth.io/cardano-prime)) |
| Audit results (Sept 2026) | Score **51.3/100** vs Ethereum 96.3 and Sui 72.3, across 25 categories. **Eight principal gaps** were reported, including bridging, wallet functionality, money markets and automated liquidity management; secondary coverage also lists perps, oracles and lending ([CoinTurk](https://en.coin-turk.com/cardano-gains-7-5yuzde-audit-reveals-defi-weaknesses-as-ada-nears-0-25/), [TronWeekly](https://www.tronweekly.com/cardano-price-prime-audit-defi-gaps/)). Key quote from the X Space: "$11.4M of TVL halves yields" (yield compression) |
| **Open RFP (Sept 2026)** | Announced around 2026-09-10/12 after the audit, via an X Space with Cardano Over Coffee ([Hokanews](https://www.hokanews.com/2026/09/cardano-ada-rises-as-midnight-and.html), [ADA Signal](https://paragraph.com/@theadasignal/ada-signal-daily-delta-2026-09-13-263000)). **Four priority RFP areas:** (1) **dual-stack wallets** (EVM + Cardano); (2) an **EVM→Cardano bridge** / general bridging route from Ethereum, Base and Arbitrum; (3) **lending and vault standards**; (4) **liquidity tooling/infrastructure**. Other "institutional-grade" items in scope ([DailyCoin](https://dailycoin.com/cardanos-200m-defi-push-puts-evm-liquidity-in-focus/)): a **common executor/scooper (batcher) standard**, **eUTxO-native vaults**, a **second independent price feed**, **curated lending markets**, **concentrated liquidity**, **automated liquidity management**, **yield-bearing collateral for CDPs**, and action-based incentive campaigns with on-chain attribution. Appendix C of the proposal adds **limit-order-book hardening** and a **"DeFi Gateway SDK with custody adapters"**, and prioritizes primitives that combine eUTxO with native non-custodial liquid staking |
| RFP economics | Rule of thumb **50:1 sustained TVL per grant dollar**. Milestone payments plus transparency reports. Preference for Cardano-native teams "at equivalent confidence". Some capital may be **recoverable** (seed LP, solver loans) with return rights. Incentive eligibility criteria (TVL thresholds, audit status, transparency, protocol age) are published **≥14 days before** any campaign |
| **How to apply** | I could not find a dedicated public RFP portal or a deadline. The public intake points are the **AlphaGrowth PRIME contact form** ([alphagrowth.typeform.com/cardano-prime](https://alphagrowth.typeform.com/cardano-prime)), AlphaGrowth on X ([@alphagrowth1](https://x.com/alphagrowth1)), and the Telegram linked from the PRIME page. Cardano's X announcement is [x.com/Cardano/status/2098451270134571144](https://x.com/Cardano/status/2098451270134571144). Exact submission format and deadlines are **[UNVERIFIED]**. Press coverage called the process "murky" ([Currency Analytics](https://thecurrencyanalytics.com/altcoins/cardano-prime-and-alphagrowth-offer-50-in-grants-for-every-1-put-into-defi-292507)) |
| Expected timeline | Enacted mid-August 2026, so Phase 2 grants (the first $3M) run roughly Sep–Dec 2026. The Phase-3 release vote is around Month 4 (about December 2026), with incentives roughly Jan–Aug 2027 **[timeline inferred from the phase plan]** |
| Pentad relationship | PRIME defers to the Pentad on infrastructure and integrations. The Pentad defers to PRIME on incentive design and LP programs |

### 2.2 Cardano Critical Integrations (CCI), run by the Pentad and administered by Intersect

| Program | Amount | Status | Delivered / scope |
|---|---|---|---|
| **CCI V1** | ₳70M | Enacted December 2025 (>80% DRep support) | Five pillars: stablecoins, custody/wallets, bridges, oracles, analytics. Delivered: **USDCx** (contract to live in 84 days, launched 2026-02-27), **Pyth**, **Dune**, and **LayerZero** (in progress). **Fireblocks missed the window** and moved to V2 ([Intersect progress](https://intersectmbo.org/news/critical-integrations-technical-progress-update-and-pentad-v2), [closure report](https://cardano.org/news/2026-05-15-cardano-critical-integrations/)) |
| **CCI V2** | ₳23M | **Enacted 2026-06-23** (CC 5 yes / 2 abstain) | Year-2 costs plus 12-month maintenance for USDCx, LayerZero, Pyth and Dune, plus a **full native Fireblocks integration** ([CardanoCube](https://www.cardanocube.com/governance/gov_actions/gov_action1cp0w6zwgwpj98jtu3r2q838lgwmhs6j49l58zx4q05lx220lmzaqqztnljz)) |

**The Pentad** was IOG, Cardano Foundation, EMURGO, Intersect and the Midnight Foundation ([AlphaGrowth PRIME doc](https://alphagrowth.io/cardano-prime)). **EMURGO stepped down on 2026-07-08** to focus on recovering from the **SecondFi wallet exploit**. SecondFi was EMURGO's rebranded Yoroi; weak key generation led to about 16M ADA (about $2.4M) stolen from 374 wallets ([The Block](https://www.theblock.co/news/ecosystems/2026-07-08-cardano-founding-entity-emurgo-steps-down-pentad-governance-role-wallet-exploit-407593), [CryptoSlate](https://cryptoslate.com/cardanos-wallet-hack-exposed-the-user-layer-holding-its-on-chain-government-together/)). "Pentad V2" was still at the exploratory stage as of March 2026.

### 2.3 Other DeFi-relevant treasury actions

| Action | ADA | Outcome | What it is |
|---|---|---|---|
| Stablecoin DeFi Liquidity Budget (Info Action) | 50M | Passed >67% (September/October 2025) | Fund of ADA plus fiat-backed stables deployed to DEXes/lending. 9-person committee plus a treasury DAO (tDAO) with DReps ([RareEvo](https://rareevo.io/rare-network-news/cardano-stablecoin-proposal-reaches-threshold)) |
| → Cardano DeFi Liquidity Budget – Withdrawal 1 | 0.8M | **Enacted** (submitted 2026-03-10). An earlier 0.5M version expired in January 2026 | Legal entity plus audit. Contracts built by **UTxO Company** (Lucas, Kasey), UI by **Sidan Labs**. Deployment of the larger 50M is **[UNVERIFIED]** |
| Cardano x Draper Dragon **Orion Fund** | 50M | **Enacted** (March 2026) | $80M venture fund. Founder cohort opened June 2026, so it is a VC route for DeFi startups ([U.Today](https://u.today/most-important-vote-of-2026-cardano-community-decides-on-50-million-ada-withdrawal-to-tim-drapers)) |
| **OpenZeppelin Stack** (administered by Intersect) | 11.79M | **Voting now** (submitted 2026-09-09) | Audited **Cardano contracts library** plus Contracts Wizard, and three reference implementations: a **Liquid Staking Protocol**, **Self-Repaying Loans**, and a **Tokenized Money Market Fund**, plus a security retainer (Koios metadata) |
| Pogun (IO-backed BTCfi) | 12.29M | **Failed** (35.67% yes; expired 2026-05-24) | See §3 |
| Bifrost Road to Mainnet Ph.1 (FluidTokens + Lantr) | 12.33M | **Expired** (submitted 2026-07-04) | See §3 |
| Alchemy by Sundial × Charms | 10M | **Expired** | Cardano-native BTC treasury protocol: FIRE (BTC+) / ICE (BTC−) structured assets |
| Strike Finance Liquidity Deployment | 9M | **Expired** | Would sell ADA for USDM to seed perps liquidity |
| Global Order Book / DeFi Kernel (Dano Finance) | 3.33M | **Expired** | Open standard for batcher-less on-chain intents/orders |
| DeltaDeFi Hydra Trading Infra (Info) | 1.5M | Expired | DeltaDeFi later paused |
| IO & Midgard Labs L2 Scalability | 10.4M | Expired | Optimistic-rollup L2 |
| Anzens: Stablecoin / CNT support / fiat ramps | 4.0M | Enacted (July 2025) | USDA distribution |
| ZK Bridge (testnet) | 0.7M | Enacted (July 2025) | Generic ZK-proof bridge framework for Cardano |
| Hoskinson's "$100M ADA→BTC/stablecoin treasury swap" idea (June 2025) | — | Discussion only; no on-chain action found | ([Forklog](https://forklog.com/en/hoskinson-proposes-100-million-ada-swap-for-bitcoin-and-stablecoins/)) |

The pattern is that **DReps approved programmatic, administrator-held budgets** (CCI, PRIME, Orion) and **rejected most single-protocol liquidity or BTCfi asks** (Pogun, Bifrost, Alchemy, Strike, DeFi Kernel, DeltaDeFi). For a new builder, the practical funding routes are PRIME's grants envelope, Orion Fund, Project Catalyst, and (pending) OpenZeppelin's library.

---

## 3. Bitcoin DeFi on Cardano

### 3.1 Strategy
- Hoskinson's thesis is that Cardano's eUTxO "shares design lineage" with Bitcoin's UTXO model. That makes Cardano "the strongest technical choice for Bitcoin DeFi", and BTC users paying ADA fees creates structural ADA demand ([crypto.news](https://crypto.news/cardano-bitcoin-defi-pogun-ada-demand/)).
- On 2026-09-18, after Pogun's treasury request failed, Hoskinson said **IO will "choose the best network for each product instead of following a Cardano-first-and-forever policy."** He also said Pogun would still ship to Cardano, roughly by mid-December 2026 ([CryptoSlate](https://cryptoslate.com/charles-hoskinson-says-cardano-no-longer-comes-first-its-treasury-vote-explains-why/)).
- IO's 2026 treasury slate was cut about 50% to $38.9M and framed around Leios plus Bitcoin DeFi ([Bitcoin.com](https://news.bitcoin.com/cardanos-leios-upgrade-and-bitcoin-defi-tool-pogun-headline-input-outputs-2026-funding-slate/)).

### 3.2 BTC rails: status and trust models

| Project | Who | Mechanism / trust model | Status (Oct 2026) |
|---|---|---|---|
| **Cardinal** | Input Output, using **BitVMX** (Fairgate) | Wraps Bitcoin UTXOs (including Ordinals) as Cardano NFTs/native assets with 1:1 peg and burn-to-redeem. **1-of-n honest committee** (BitVM-style fraud proofs) | **Mainnet demo** at Bitcoin 2025 (May 2025) ([CoinDesk](https://www.coindesk.com/tech/2025/05/23/bitcoin-ordinals-can-now-be-bridged-to-cardano-through-bitvmx), [Fairgate](https://www.fairgate.io/post/15-introducing-cardinal-how-bitvmx-bridges-bitcoin-ordinals-to-cardano-unlocking-cross-chain-defi)). Described as IO's "open-source Bitcoin bridge specification". No production BTC liquidity found **[UNVERIFIED]** |
| **Pogun** | IO-backed team led by Omer Husain (built Cardinal); Bo Zhang (ex-Grayscale COO) advising | (1) **Non-margin, oracle-free credit market**: bilateral fixed-term loans, collateral at risk only on definitive default, transferable **Bond Tokens**. (2) Yield dApp. (3) **BitVM bridge** that moved from BitVM3 to a **BABE**-based design with recursive Halo2 proofs over **Mithril** certificates. 1-of-N verifiers, "trust-minimized, not trustless". Midnight is used for privacy ([IOG](https://www.iog.io/news/why-pogun-bitcoin-defi-architecture-is-achievable), [Momentum](https://momentum.cardano.iog.io/proposals/pogun)) | **Not live.** Missed its Q2 2026 credit-market target. Treasury request failed. Site says "coming soon" ([Coindoo](https://coindoo.com/cardanos-bitcoin-defi-push-faces-its-first-delivery-test/)). Hoskinson's new target is about mid-December 2026 |
| **Bifrost** | FluidTokens, Lantr, zkFold (Catalyst F14) | **SPO-secured** bridge: Cardano SPOs collectively control the BTC multisig/threshold. Watchtowers relay BTC headers. Merkle inclusion proofs, with ZK used for SPO-misbehavior proofs. Mints **fBTC**. Plans both federated and SPO-threshold custody modes ([BEX/BlockEden](https://bex.co/blog/2026/01/26/bitcoin-cardano-bifrost-bridge-fluidtokens-btcfi), [repo](https://github.com/FluidTokens/ft-bifrost-bridge)) | **BTC testnet since 2026-04-16.** Peg-in→fBTC mint flow demoed 2026-04-25. ₳12.3M mainnet-readiness request expired. Plan was audited mainnet under controlled access by about March 2027 |
| **BitcoinOS Grail** | BitcoinOS (Edan Yago) | ZK (BitSNARK) bridge | Demo transfer to Cardano in 2024–25. **No evidence of production Cardano support as of 2026** [UNVERIFIED] ([CryptoSlate](https://cryptoslate.com/cardano-unlocks-bitcoin-liquidity-with-bitcoinos-grail-bridge-integration/)) |
| **Rosen Bridge** | Ergo community | Watchers plus guard federation secured via Ergo. Supports Bitcoin, Cardano, Ethereum, BSC, Doge | Live, small ($0.32M TVL on Cardano) |
| **Wanchain** | Wanchain | Storeman MPC federation | Live. Brings BTC, ETH, USDC and USDT to Cardano. Small ($0.76M) ([cexplorer](https://cexplorer.io/article/wanchain-bridge-connects-cardano-with-many-l1s-and-l2s)) |
| **Indigo iBTC** | Indigo | **Synthetic**: ADA-collateralized CDP tracking the BTC price via oracle. No BTC backing | Live since 2022 |
| **Alchemy (FIRE/ICE)** | Sundial × Charms | BTC-backed structured long/short exposure | Treasury request expired. Status **[UNVERIFIED]** |
| anetaBTC (cBTC) | anetaBTC | Federated wrapped BTC | Legacy/archived on CardanoCube **[UNVERIFIED current state]** |

**Related UX primitives that work today:**
- **FluidTokens BTC/ETH accounts**: Aiken contracts let a **Bitcoin or Ethereum wallet (Xverse, Unisat, Leather, MetaMask) control a Cardano address** through ECDSA signature verification. Third parties submit the transaction and get a small ADA tip. There is a live mainnet demo ([repo](https://github.com/FluidTokens/ft-cardano-btc-eth-accounts)). This is the closest thing Cardano has to account abstraction for non-Cardano users.
- **Aquarium / babel fees** (FluidTokens): dApps fund "FeeTanks" so users can pay fees in any native token, for example bridged BTC ([cexplorer](https://cexplorer.io/article/cardano-will-allow-paying-fees-with-tokens)). Protocol-level **Babel Fees** and CIP-159 are in IO's funded "Cardano Upgrades" workstream (enacted April 2026).

**Bottom line (late 2026):** no trust-minimized BTC bridge is in production on Cardano. Real BTC liquidity on Cardano is in the low single-digit millions at most **[UNVERIFIED; no DefiLlama BTC-asset breakdown found]**.

---

## 4. Interop & bridges

| Rail | Status | Notes |
|---|---|---|
| **USDCx (Circle xReserve)** | **Live** since 2026-02-27 | Ethereum USDC ↔ Cardano USDCx. IOG paid bridging fees for 90 days (to 2026-05-28). Bridge-and-swap into any CNT via Minswap |
| **LayerZero v2** | **Phased rollout.** Announced 2026-02-12 (Consensus HK) | Phases: endpoints → **Stargate** liquidity → dev tools → product integrations. Goal: "any asset can deploy on Cardano by end-2026", 800+ tokens ([IOG](https://www.iog.io/news/cardano-is-connecting-to-all-of-crypto), [cardano.org](https://cardano.org/news/2026-06-08-cardano-is-connecting-to-all-of-crypto/)). Note: LayerZero's public deployments metadata (`metadata.layerzero-api.com/v1/metadata/deployments`) had **no Cardano entry on 2026-10-07**, so I could not confirm a public mainnet endpoint **[UNVERIFIED]**. eUTxO forced a redesign of standard endpoint/OFT patterns |
| **Wormhole** | **Proposal stage only** | $5M treasury ask discussed on the forum. Covers NTT for native multichain stablecoins (M0), ADA on Ethereum and Solana within 60 days, and RWA links (Securitize, DigiFT). AMA held 2026-06-08 ([Coinfomania](https://coinfomania.com/it/wormhole-teams-up-with-cardano-cf-for-ama-on-cross-chain-deployment/)). **No on-chain governance action found on Koios as of 2026-10-07** |
| **Axelar** | Nothing announced | — |
| **IBC** | **Testnet**: Cardano Preprod ↔ Injective testnet (August 2026) | Escrow-and-mint IBC vouchers, with modules translating eUTxO ↔ account model ([The Crypto Basic](https://thecryptobasic.com/2026/08/04/cardano-expands-interoperability-as-first-ibc-integration-with-injective-goes-live/), [cardano-ibc-incubator](https://github.com/cardano-foundation/cardano-ibc-incubator)) |
| **Milkomeda C1 (EVM sidechain)** | **Shut down** | ([forum](https://forum.cardano.org/t/current-status-of-cardano-evm-interoperability-milkomeda-rosen/153432)) |
| **Wanchain**, **Rosen**, **ChainPort**, **NEAR Intents** | Live, small | NEAR Intents gives intent-based cross-chain swaps that include ADA |
| **Fireblocks** | Native integration funded in CCI V2. "Announced plans" for full CNT support | Matters for institutional LPs and market makers |

---

## 5. Builder angle

### 5.1 Composable, open components

| Component | What you get | Link |
|---|---|---|
| **Minswap SDK** | Pool price feeds, history, trade-price/impact calculation, order build and submit (Lucid), syncer. Covers AMM V1/V2, Stableswap, LBE V2. V2 deployed on **Preprod** | [github.com/minswap/sdk](https://github.com/minswap/sdk), [docs](https://docs.minswap.org/), [testnet-preprod.minswap.org](https://testnet-preprod.minswap.org) |
| **SundaeSwap** | SDK plus V3 contracts. Preview-network app | [sundae-sdk](https://github.com/SundaeSwap-finance/sundae-sdk), [sundae-contracts](https://github.com/SundaeSwap-finance/sundae-contracts), [app.preview.sundae.fi](https://app.preview.sundae.fi) |
| **Splash** | Core contracts, protocol SDK, off-chain execution agents (anyone can run an executor) | [splash-core](https://github.com/splashprotocol/splash-core), [protocol-sdk](https://github.com/splashprotocol/protocol-sdk), [green-order-offchain-agent](https://github.com/splashprotocol/green-order-offchain-agent) |
| **WingRiders** | V2 DEX contracts | [dex-v2-contracts](https://github.com/WingRiders/dex-v2-contracts) |
| **Indigo** | `indigo-sdk`, **`dexter`** (multi-DEX swap library), plus **`indigo-mcp` / `cardano-mcp`** (AI-agent tooling) | [IndigoProtocol on GitHub](https://github.com/IndigoProtocol) |
| **Strike Finance** | Perpetuals, options and forwards contracts, a Hummingbot connector, `strike-builder-reference`, **agent "skills"** | [github.com/strike-finance](https://github.com/strike-finance) |
| **DexHunter** | Aggregator **partner API** for swaps, limit orders and DCA. `X-Partner-Id` header; base `https://api-us.dexhunterv3.app`; rev-share | [API docs](https://dexhunter.gitbook.io/dexhunter-partners/api-reference/api) |
| **Liqwid** | Libraries (`liqwid-libs`), Agora governance | [Liqwid-Labs](https://github.com/Liqwid-Labs). A public lending SDK was **not found** [UNVERIFIED] |
| **FluidTokens** | Loans v4, Aquarium (fee abstraction), BTC/ETH-key accounts, CIP-113 (programmable tokens), Bifrost bridge | [github.com/FluidTokens](https://github.com/FluidTokens) |
| **Genius Yield** | Order-book contracts plus API server | [dex-contracts-api](https://github.com/geniusyield/dex-contracts-api) |
| **DeltaDeFi** | TS SDK for the Hydra order book (project paused) | [typescript-sdk](https://github.com/deltadefi-protocol/typescript-sdk) |
| **Oracles** | Pyth Lazer (Aiken plus TS), Charli3 ODV client, Orcfax feeds | See §1.4 |
| **DeFi Kernel** | Open standard and registry for **batcher-less** orders/intents: permissionless fills, published datum/redeemer schemas, discoverable via CIP-89 beacon tokens or deterministic addresses | [defikernel.org](https://defikernel.org) |
| **Design-pattern libs** | Anastasia Labs Aiken design patterns (stake-validator "withdraw-zero" forwarding, UTxO indexers, merkelized validators, etc.) | [aiken-design-patterns](https://github.com/Anastasia-Labs/aiken-design-patterns) |

### 5.2 eUTxO DeFi design patterns (what EVM devs need to unlearn)
1. **Order-UTxO plus batcher ("scooper"/executor).** Users lock an order UTxO and an off-chain batcher matches it against the pool UTxO. This solves the "everyone spends the same pool UTxO" contention problem, but it adds batcher trust, fees and latency, and every DEX runs its own. PRIME's call for a **common executor/scooper standard** targets this fragmentation.
2. **Order books on L1.** Each order is its own UTxO and fills are atomic and parallel (Genius Yield, Splash, Dano/DeFi Kernel). This fits eUTxO naturally, but needs good indexing and discovery (beacon tokens, CIP-89).
3. **Intents without batchers.** The DeFi Kernel approach: any party fills any order directly, so solvers and keepers compete. The solver network itself is mostly missing.
4. **Pull oracles via reference inputs / withdraw-zero.** Pyth verifies signed updates in a stake-withdrawal script, and consumers read the result as a reference input in the same transaction. The consumer enforces freshness.
5. **L2 for latency-sensitive trading.** Hydra heads (DeltaDeFi, Strike-style matching) suit HFT/order books but carry operational and funding risk; DeltaDeFi paused. Midgard (optimistic rollup) L2 funding failed in 2026.
6. **Fee/UX abstraction.** Babel-fee-style liabilities (FluidTokens Aquarium today, protocol-level Babel Fees in progress) plus foreign-key accounts (BTC/ETH signatures) stand in for account abstraction.
7. **Throughput.** Ouroboros Leios is on the public testnet "Musashi" since 2026-06-23 (about 6× Praos throughput under synthetic load). Mainnet is targeted for late 2026 via the Dijkstra hard fork **[subject to governance]** ([Essential Cardano weekly report](https://www.essentialcardano.io/development-update/weekly-development-report-as-of-2026-08-21)). PV11 "van Rossem" hard fork was enacted in June 2026 (Koios).

### 5.3 Primitive gap map: Ethereum/Solana vs Cardano

| Primitive | ETH/SOL state of the art | Cardano today | Gap |
|---|---|---|---|
| Spot AMM | Uniswap v3/v4 CL, hooks; Orca/Meteora DLMM | Constant-product plus stableswap. CL is only partial (Splash, Sundae V3 fee tiers) | **Concentrated liquidity plus ALM vaults** (named in PRIME) |
| Money markets | Aave, Morpho curated/isolated vaults, Kamino | Liqwid (pooled), Surf (isolated), FluidTokens P2P | **Curated vaults, vault standard (4626-like)**, yield-bearing collateral |
| Perps | Hyperliquid, dYdX, Jupiter, Drift | Strike (about $2M TVL). DeltaDeFi paused | Depth, market makers, oracle redundancy |
| Stablecoins | USDT/USDC native, USDe, PYUSD | USDCx (xReserve), USDM, USDA, DJED, iUSD, USDr (new) | **No native USDT**. No yield-bearing delta-neutral stable |
| BTCfi | WBTC, cbBTC, tBTC, Babylon | iBTC (synthetic), small federated bridges | **No production trust-minimized BTC** |
| LST / restaking | Lido, Jito, EigenLayer | Native liquid staking. Locked-in-contract ADA often loses staking | Liquid-staking primitive for contract-locked ADA (OpenZeppelin reference impl pending) |
| Intents / solvers | UniswapX, CoW, 1inch Fusion, Jupiter | Batchers per DEX. DeFi Kernel standard. NEAR Intents support | **Shared solver/executor network** |
| AA / wallet UX | ERC-4337, 7702, smart wallets, passkeys | Native multisig/scripts. FluidTokens BTC/ETH-key accounts. Babel fees WIP | **Dual-stack wallet**, gas abstraction at scale |
| Bridges | LayerZero/CCIP/Wormhole everywhere | USDCx live. LayerZero phased. IBC on testnet | General EVM→Cardano route (PRIME RFP) |
| RWA / tokenized funds | BUIDL, BENJI, Ondo | USDr (new), USDM reserves | Tokenized MMF reference impl (OpenZeppelin, pending) |
| Prediction markets | Polymarket | Bodega (about $0) | Wide open |
| Analytics | Dune, Flipside, Allium | **Dune now live** (CCI) | Better DeFi dashboards and attribution (PRIME needs on-chain attribution) |

---

## 6. Things to try on testnet

| What | Network | How |
|---|---|---|
| Test ADA | Preprod / Preview | Cardano testnet faucet: [docs.cardano.org/cardano-testnets/tools/faucet](https://docs.cardano.org/cardano-testnets/tools/faucet) |
| Minswap V2 swaps/LP plus SDK | **Preprod** | [testnet-preprod.minswap.org](https://testnet-preprod.minswap.org) with [`minswap/sdk`](https://github.com/minswap/sdk) |
| SundaeSwap | **Preview** | [app.preview.sundae.fi](https://app.preview.sundae.fi) with `sundae-sdk` |
| Strike Finance perps | Testnet front-end | `testnet.strikefinance.org` responds. That it is a full testnet deployment is **[UNVERIFIED]** |
| USDCx | Testnet asset `asset1ejelsh8crza8dyghxzsjhkjqutzr7q3dnregng` | Pair with Ethereum Sepolia USDC via the xReserve docs. Whether testnet bridge mode exists in the public UI is **[UNVERIFIED]** |
| Pyth Pro price updates | **Preprod** | Pyth docs' Cardano example uses preprod. Needs a Lazer access token |
| Charli3 ODV oracle | **Preprod** plus mainnet | [`charli3-pull-oracle-client`](https://github.com/Charli3-Official/charli3-pull-oracle-client) |
| Bifrost BTC bridge (fBTC) | Bitcoin testnet ↔ Cardano testnet | [ft-bifrost-bridge](https://github.com/FluidTokens/ft-bifrost-bridge). Public testnet since April 2026 |
| Cardano ↔ Injective IBC | Preprod ↔ Injective testnet | [cardano-ibc-incubator](https://github.com/cardano-foundation/cardano-ibc-incubator) |
| DexHunter API | Mainnet (partner key) | Testnet support **[UNVERIFIED]** |

---

## 7. Opportunity gaps for new builders (ranked by funding tailwind)

1. **eUTxO vault standard plus automated liquidity management (ALM).** A 4626-equivalent vault interface and keeper-run strategy vaults that rebalance LP positions across Minswap, Splash and Sundae, with yield-bearing receipt tokens usable as Indigo/Liqwid collateral. This is named explicitly in the PRIME RFP (vault standards, ALM, concentrated liquidity, yield-bearing CDP collateral).
2. **Curated/isolated money markets on USDCx plus Pyth Pro** (Morpho-style). Risk curators, isolated markets, fixed-term markets that reuse Pogun-style non-margin bond tokens, and a second oracle adapter (Pyth plus Charli3/Orcfax median) to answer the "second price feed" gap.
3. **EVM-native on-ramp ("dual-stack" UX).** MetaMask-controlled Cardano accounts (extend FluidTokens' ECDSA accounts), gas abstraction via babel fees, and one-click EVM→Cardano routing once LayerZero/Stargate goes live. This sits directly on two of the four PRIME RFP areas.
4. **Shared executor/solver network for intents.** A neutral batcher/solver market implementing the DeFi Kernel / CIP-89 order discovery, so DEXes stop running siloed batchers. PRIME names a "common executor/scooper standard". The DeFi Kernel treasury ask failed, so the space is open.
5. **BTC-ready credit and yield primitives.** Build BTC-collateralized USDCx lending, BTC covered calls, and structured products against iBTC/fBTC interfaces now, so they are ready to plug into Bifrost or Pogun's bridge when either reaches mainnet. Be honest about the trust model in the UI (synthetic vs federated vs SPO-threshold vs BitVM 1-of-N).

Honorable mentions: prediction markets (empty category), perps market-making infrastructure for Strike (Hummingbot connector exists), and Dune dashboards plus on-chain attribution tooling that PRIME needs for its TVL verification.

**Risks to flag to readers:** TVL and liquidity are thin, which means high slippage and yield compression (PRIME: "$11.4M of TVL halves yields"). Treasury politics are volatile (several high-profile asks failed). IO no longer commits to Cardano-first. Wallet-layer security incidents have happened (SecondFi). LayerZero mainnet availability could not be independently confirmed.

---

## Sources

**Data APIs (pulled 2026-10-07)**
- https://api.llama.fi/v2/chains · https://api.llama.fi/protocols · https://api.llama.fi/v2/historicalChainTvl/Cardano · https://api.llama.fi/overview/dexs/Cardano · https://stablecoins.llama.fi/stablecoins · https://coins.llama.fi/prices/current/coingecko:cardano
- https://api.koios.rest/api/v1/proposal_list (governance actions and metadata)
- https://metadata.layerzero-api.com/v1/metadata/deployments (no Cardano entry found)

**PRIME**
- https://alphagrowth.io/cardano-prime
- https://alphagrowth.typeform.com/cardano-prime
- https://danonight.com/news/cardano-prime-treasury-withdrawal
- https://thecryptobasic.com/2026/08/12/cardano-community-approves-120-million-ada-allocation-to-boost-defi-liquidity/
- https://en.coin-turk.com/cardano-gains-7-5yuzde-audit-reveals-defi-weaknesses-as-ada-nears-0-25/
- https://www.tronweekly.com/cardano-price-prime-audit-defi-gaps/
- https://dailycoin.com/cardanos-200m-defi-push-puts-evm-liquidity-in-focus/
- https://www.hokanews.com/2026/09/cardano-ada-rises-as-midnight-and.html
- https://paragraph.com/@theadasignal/ada-signal-daily-delta-2026-09-13-263000
- https://github.com/shiodome47/cardano-note/pull/3
- https://thecurrencyanalytics.com/altcoins/cardano-prime-and-alphagrowth-offer-50-in-grants-for-every-1-put-into-defi-292507
- https://en.coin-turk.com/cardano-prime-launches-1-for-50-defi-push-ada-trades-at-0-2091/
- https://x.com/Cardano/status/2098451270134571144
- https://x.com/Cardano_CF/status/2087164970353979629

**Critical Integrations / Pentad / treasury**
- https://intersectmbo.org/news/critical-integrations-technical-progress-update-and-pentad-v2
- https://cardano.org/news/2026-05-15-cardano-critical-integrations/
- https://www.cardanocube.com/governance/gov_actions/gov_action1cp0w6zwgwpj98jtu3r2q838lgwmhs6j49l58zx4q05lx220lmzaqqztnljz
- https://rareevo.io/rare-network-news/cardano-stablecoin-proposal-reaches-threshold
- https://u.today/most-important-vote-of-2026-cardano-community-decides-on-50-million-ada-withdrawal-to-tim-drapers
- https://forklog.com/en/hoskinson-proposes-100-million-ada-swap-for-bitcoin-and-stablecoins/
- https://www.theblock.co/news/ecosystems/2026-07-08-cardano-founding-entity-emurgo-steps-down-pentad-governance-role-wallet-exploit-407593
- https://cryptoslate.com/cardanos-wallet-hack-exposed-the-user-layer-holding-its-on-chain-government-together/
- https://www.cardanocube.com/governance/gov_actions/gov_action1suskjc6c4nw58c6wtmv77xe79gwj47wp4gvh9cqhhxujwxmam3cqqkz5nwj?tally=live

**Stablecoins / interop**
- https://www.circle.com/blog/usdcx-on-cardano-now-available-via-circle-xreserve
- https://www.iog.io/news/usdcx-on-cardano-here-s-what-happened-in-the-first-seven-days
- https://usdcx.iog.io/bridge
- https://www.iog.io/news/cardano-is-connecting-to-all-of-crypto
- https://cardano.org/news/2026-06-08-cardano-is-connecting-to-all-of-crypto/
- https://coinfomania.com/it/wormhole-teams-up-with-cardano-cf-for-ama-on-cross-chain-deployment/
- https://thecryptobasic.com/2026/08/04/cardano-expands-interoperability-as-first-ibc-integration-with-injective-goes-live/
- https://forum.cardano.org/t/current-status-of-cardano-evm-interoperability-milkomeda-rosen/153432
- https://cexplorer.io/article/wanchain-bridge-connects-cardano-with-many-l1s-and-l2s
- https://www.kucoin.com/news/flash/realfi-launches-cardano-native-stablecoin-backed-by-real-world-assets
- https://thecryptobasic.com/2026/10/05/what-the-hell-else-can-i-do-cardano-founder-defends-ada-progress
- https://medium.com/@USDMOfficial/the-shift-to-independent-stability-why-usdm-is-the-new-b2b-treasury-standard-on-cardano-eddb7f2a5fd5

**BTCfi**
- https://crypto.news/cardano-bitcoin-defi-pogun-ada-demand/
- https://www.iog.io/news/why-pogun-bitcoin-defi-architecture-is-achievable
- https://momentum.cardano.iog.io/proposals/pogun
- https://coindoo.com/cardanos-bitcoin-defi-push-faces-its-first-delivery-test/
- https://cryptoslate.com/charles-hoskinson-says-cardano-no-longer-comes-first-its-treasury-vote-explains-why/
- https://news.bitcoin.com/cardanos-leios-upgrade-and-bitcoin-defi-tool-pogun-headline-input-outputs-2026-funding-slate/
- https://www.coindesk.com/tech/2025/05/23/bitcoin-ordinals-can-now-be-bridged-to-cardano-through-bitvmx
- https://www.fairgate.io/post/15-introducing-cardinal-how-bitvmx-bridges-bitcoin-ordinals-to-cardano-unlocking-cross-chain-defi
- https://bex.co/blog/2026/01/26/bitcoin-cardano-bifrost-bridge-fluidtokens-btcfi
- https://github.com/FluidTokens/ft-bifrost-bridge
- https://github.com/FluidTokens/ft-cardano-btc-eth-accounts
- https://cexplorer.io/article/cardano-will-allow-paying-fees-with-tokens
- https://cryptoslate.com/cardano-unlocks-bitcoin-liquidity-with-bitcoinos-grail-bridge-integration/

**Protocols / developer docs**
- https://docs.pyth.network/price-feeds/pro/integrate-as-consumer/cardano
- https://developers.cardano.org/tools/pyth-pro/
- https://github.com/pyth-network/pyth-lazer-cardano
- https://docs.charli3.io/oracles/products/pull-oracle/summary
- https://github.com/Charli3-Official/charli3-pull-oracle-client
- https://github.com/orcfax/cer-feeds
- https://developers.cardano.org/docs/build/integrate/oracles/orcfax
- https://github.com/minswap/sdk · https://docs.minswap.org/
- https://github.com/SundaeSwap-finance/sundae-sdk · https://github.com/SundaeSwap-finance/sundae-contracts
- https://github.com/splashprotocol/splash-core · https://cardano.org/apps/splash/
- https://github.com/WingRiders/dex-v2-contracts
- https://github.com/IndigoProtocol
- https://github.com/strike-finance
- https://dexhunter.gitbook.io/dexhunter-partners/api-reference/api
- https://github.com/Liqwid-Labs
- https://github.com/geniusyield/dex-contracts-api
- https://github.com/deltadefi-protocol/typescript-sdk · https://defillama.com/protocol/deltadefi
- https://defikernel.org
- https://github.com/Anastasia-Labs/aiken-design-patterns
- https://github.com/cardano-foundation/cardano-ibc-incubator
- https://cardano.org/apps/surf-lending/
- https://docs.cardano.org/cardano-testnets/tools/faucet
- https://testnet-preprod.minswap.org · https://app.preview.sundae.fi

**Macro context**
- https://cointelegraph.com/news/defi-tvl-falls-39-2026-erases-45b-value
- https://www.essentialcardano.io/development-update/weekly-development-report-as-of-2026-08-21
