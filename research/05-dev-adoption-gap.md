# 05: The Cardano Developer-Adoption Gap: Diagnosis, Lessons from Other Chains, and What Our Site Should Do

*Research date: 2026-10-07. On-chain numbers were pulled live from DefiLlama, Koios and CoinGecko on 2026-10-06/07. Any claim I could not confirm is marked **[UNVERIFIED]**. Numbers in brackets like [S12] point to the Sources list at the end.*

---

## TL;DR

- **The gap is real, and in 2026 it is getting wider.** Electric Capital's 2024 data, as cited by the Cardano Foundation, gives Cardano about 672 monthly active developers, 276 of them full-time, which ranks #15 [S1]. Ethereum and Solana each added more than 11k new developers in the first nine months of 2025 [S3]. IO's own 2026 proposal says Cardano has "around 17x fewer developers than Ethereum" and that "the gap keeps widening" while Cardano's developer count stays flat [S10].
- **Cardano does well on commit counts and badly on app-layer adoption.** It ranks #6 by GitHub activity in Santiment's Q1 2026 list [S6]. But most of those commits come from core-protocol R&D (Leios, Plutus, Mithril), not from independent app builders. The CF 2025 developer survey drew only 109 respondents [S8].
- **On-chain demand is very thin.** TVL is $73M (#36 on DefiLlama, down from $573M in Dec 2024). Stablecoins total $67M (#40). Transactions run at about 20–32k a day, down from about 72k a day in Nov 2024. DEX volume is about $4M a day [S20–S23]. Visible 2026 failures: JPG Store closed (May 23), TapTools wound down (June 3), DeltaDeFi paused, and Hoskinson warned of "a wave of failures" [S27–S30].
- **Developers name the same pain points every year:** fragmented tooling, no "blessed path", hard off-chain transaction building, the eUTxO learning curve, and few funding routes beyond Catalyst. These come verbatim from the CF 2025 survey [S9].
- **Several old objections no longer hold.** "You need Haskell" is outdated: Aiken is used by more than 75% of surveyed developers [S8]. "No stablecoin, oracle or bridge" is outdated: USDCx, Pyth, LayerZero and Dune all went live in 2026 [S31]. "No docs" is mostly outdated: the 2026 developer portal has a 7-module curriculum, one-command templates, a contracts library, a "Cardano for Ethereum Developers" guide and AI skills [S12, S13].
- **Still unaddressed: opportunity discovery, motivation, and the pipeline from learning to earning to funding.** Solana (Superteam Earn + Colosseum) and Ethereum (Speedrun + ETHGlobal) win on these, not on docs. A community member already compiles "Cardano Weekly Opportunities" by hand because opportunities are "spread across different websites, forums, job boards, and social media" [S35].
- **Recommendation:** don't build yet another docs site. Build the **conversion and opportunity layer** on top of the official portal:
  1. An honest "should I build here?" page backed by live data.
  2. A persona router with "Rosetta stone" translations (Solidity → Aiken, Anchor → Aiken).
  3. A **live opportunity board** (funding, RFPs, bounties, hackathons, jobs, ecosystem gaps).
  4. On-chain-verified quests that feed the board.
  5. A showcase of solo and indie builders.

  Position Cardano for 2026 as **"the chain whose treasury pays builders, with UTXO-native rails for Bitcoin, privacy (Midnight) and agent payments (x402)"**. Do not position it as a DeFi-liquidity play.

---

## 1. The data

### 1.1 Developer counts (Electric Capital and others)

Methodology matters a great deal here. Electric Capital counts unique authors of commits to ecosystem repositories. Artemis counts weekly active developers. Santiment, Token Terminal and Chainspect mostly count commits. Cardano looks very different depending on which one you use.

| Metric | Cardano | Ethereum | Solana | Base | Sui | Aptos | Bitcoin | Source |
|---|---|---|---|---|---|---|---|---|
| Monthly active devs (EC 2024 report) | **672** (276 full-time), rank **#15** | 6,244 (single chain); 31,869 total active (2025) | 3,201; 17,708 total active (2025) | 1,140 (L2 MAD); 4,287 "active devs" | n/a (>1k new devs in 2024) | 125 full-time (+10.6% YoY) | ~1,200; 11,036 total contributors | [S1][S2][S4][S5] |
| New devs, Jan–Sep 2025 (EC 2025) | not reported in public coverage **[UNVERIFIED]** | **16,181** | **11,534** | — | — | — | **7,494** | [S3] |
| New devs in 2024 (EC) | Not on the list of ecosystems that gained >1,000 new devs (Aptos, Base, Bitcoin, ICP, NEAR, Polkadot, Polygon, Starknet, Sui were) → implies <1k **[inference]** | — | **7,625** (+83%), #1 for newcomers | >1k | >1k | >1k | >1k | [S2] |
| Weekly active devs (Cryptometheus, mid-2025) | **175** (−33% QoQ) | — | 499 | — | — | — | — | [S7] |
| Weekly active devs (Artemis, Mar 2026) | not reported | 2,811 (−34%) | 942 (−40%) | 378 (−52%) | — | −60% | — | [S5] |

Industry context:
- Crypto monthly active developers peaked around 31k in 2022 and fell to 23,613 in 2024 (−7% YoY). There are 7,896 full-time developers globally. Developers with 2+ years of tenure grew 27% and now write about 70% of commits [S4][S5].
- Since early 2025, weekly crypto commits are down about 75% (850k → 210k) and weekly active devs are down about 56% (to ~4,600) as talent moves to AI [S5].
- Cardano is competing in a shrinking pool. Pulling in the developers who remain (who are mostly experienced and multichain) depends on clear economic and technical reasons, not general awareness.

### 1.2 The commit paradox: "top-3 by GitHub activity" yet few app builders

- Santiment Q1 2026 ranks Cardano **#6** by GitHub activity, focused on the "Protocol 11 hard fork + retail payments" [S6].
- Token Terminal: 233 commits/week, about 6.2% of all L1 commits [S24].
- Chainspect lists Cardano #6 with 3,784 "active developers" against Ethereum 12,170, Solana 10,945, Sui 1,500 and Base 1,497. The time window shown appears to be "all-time" [S25].
- **Interpretation:** these counts are dominated by core-protocol work funded by the treasury and the founding entities (Leios prototypes, Plutus built-ins, Mithril, formal methods in Lean 4 [S26]). Independent app-layer developers are much fewer. The CF survey is the best direct sample of that group, and it reached 109 respondents [S8]. **The website should never use commit rankings as its pitch.** Experienced developers know what those numbers measure, and leading with them reads as cope.

### 1.3 On-chain usage (live, 2026-10-06/07)

| Metric | Value | Rank / trend | Source |
|---|---|---|---|
| DeFi TVL | **$73.1M** | **#36** of all chains (Sui $550M #15, Near $237M #20, Aptos $55M #40) | DefiLlama API [S20] |
| TVL history | $573M (Dec 2024) → $352M (Sep 2025) → $142M (Mar 2026) → $61M (Sep 2026) → $73M | −87% from Dec 2024 peak | [S21] |
| Stablecoin supply on Cardano | **$67.4M** | #40 (Aptos $1.05B, Sui $483M) | [S22] |
| USDCx (launched 2026-02-27) | ~$17.5M supply at launch, ~36% of Cardano stablecoins | first Tier-1 stablecoin on Cardano | [S31] |
| DEX volume | ~$4.0M / 24h; ~$82M / 30d | — | [S23] |
| Transactions | ~18–32k/day (epochs 653–659: 89k–160k per 5-day epoch) | ~72k/day in Nov 2024 (epoch 520: 362k/epoch) | Koios [S19] |
| Daily active addresses | ~13.3k | ~13.7k in July 2026 | DefiLlama via [S20][S28] |
| Fees | ~53.5k ADA per 5-day epoch (~$2.9k/day) | — | Koios totals [S19] |
| On-chain treasury | **~1.371B ADA (~$366M at $0.267)** | among the largest on-chain treasuries | Koios totals [S19], CoinGecko [S38] |
| ADA price / market cap | $0.267 / ~$10.0B | was $0.22 on 2026-06-03 | [S38][S29] |
| Projects "building on Cardano" (Essential Cardano count) | ~2,005–2,009 (2025) | a cumulative, loosely defined count; Cardano Cube curates 650+ | [S39][S40] |

**2026 ecosystem events that shape how developers see Cardano:**
- JPG Store (the main NFT marketplace) shut down permanently on May 23.
- TapTools (analytics/API) began winding down on June 3 after losing its co-founders, COO and CTO.
- Hoskinson said "There's going to be a wave of failures" and that older projects are "no longer in an investable state". He then posted "I'm taking a break" [S27][S28][S29].
- DeltaDeFi (a Hydra-based exchange) paused operations, and SBI chose Solana in Japan [S36].
- Milkomeda, the EVM sidechain, shut down on Sept 1, 2025 [S41].
- A May 2026 treasury fight pitted developers wanting "measurable business outcomes" against IO's 32.9M ADA research proposal, which faced more than 70% opposition before the vote closed. The final outcome is **[UNVERIFIED]** [S32][S33].

**Positive 2026 signals:**
- USDCx, LayerZero, Pyth and Dune all launched under the 70M-ADA Critical Integrations program [S31].
- Midnight mainnet launched (around end of March 2026) [S42].
- The Van Rossem hard fork (PV11) went live on 2026-07-18, decided fully on-chain. The Dijkstra hard fork, bringing Linear Leios, nested transactions and Plutus V4, is tentatively set for Dec 2026–Mar 2027 [S43].
- The 50M-ADA Orion venture fund (Draper Dragon) was approved [S44].
- Cardano was ranked #6 by GitHub activity in Q1 2026 [S6].

### 1.4 Cardano-specific developer surveys

**Cardano Foundation "State of the Cardano Developer Ecosystem" 2025** (4th edition, published 2025-12-11, n=109) [S8][S9]:
- Languages: TypeScript 63.3%, JavaScript 55.0%, Python 43.1%, **Aiken 34.8%** (#2 among junior devs), Haskell 29.3%, Rust 25.6%.
- Smart-contract languages: Aiken is used by more than 75%. Plu-ts and Scalus are around 9% each, Plinth/PlutusTx around 8%, OpShin around 7%, Helios around 6%.
- Median satisfaction with the smart-contract ecosystem is **7/10** (IQR 6–8).
- 30.2% of respondents are new or early in functional programming. About 28% have 2 years or less of blockchain experience.
- Most attractive app categories: Identity & Authentication (broad appeal); DeFi and core infrastructure (strongest consensus). Top roadmap asks: **Leios throughput** and **better documentation**.
- **Biggest pain points (verbatim CF categories):**
  - "Immature, fragmented, or incomplete libraries across languages."
  - "Lack of plug-and-play libraries compatible with prototyping workflows and LLM coding."
  - "Poor off-chain/on-chain integration; fractured tooling ecosystems."
  - "Fragmented documentation scattered across sources with no single 'blessed path'."
  - "Onboarding tools are insufficient; new developers struggle to know what to use."
  - "Difficulty going from zero to a working dApp in any environment."
  - "Overall poor DX compared to other ecosystems (especially EVM)."
  - "Fear/hesitance from VCs and limited funding pathways beyond Catalyst."
  - "Uncertainty around Cardano's positioning and market direction."
  - "Difficulty finding up-to-date information without relying on Twitter."
  - "General chaos."
- CF's own summary: *"In contrast to ecosystems like Ethereum—where complexity is often hidden behind opinionated frameworks—Cardano developers still shoulder much of the architectural burden directly."*
- **What developers value most** (useful material for positioning): community and culture, "transaction determinism provided by the eUTxO model", "Cardano Native Assets as first-class citizens", "a treasury-funded ecosystem that empowers contributors", Aiken, and the infrastructure from TxPipe, Ogmios and Mesh.

**Intersect "State of Developer Experience" survey.** It ran in 2025, with a 2026 update opened in Feb 2026. Intersect also runs a DevEx Working Group (bi-weekly) and a Developer Advocate Program [S16][S17]. Published 2026 results: **[UNVERIFIED / not found]**.

**IO "Developer Experience Initiative"** (3.6M ADA from the treasury, June–Dec 2026) [S10][S11]:
- Problem statement: *"fragmented documentation, unmaintained tools and libraries, and no clear onboarding path… Building is too hard, so most developers abandon or switch to a different blockchain."*
- Deliverables:
  - `cardano-init` scaffolding CLI (end of Q3 2026)
  - an OpenZeppelin-style ContractsLibrary with 5 audit-ready contracts (beta Dec 2026)
  - a Developer HUB with **EVM-developer, Web2-developer and technical-entrepreneur personas**
  - bounties for tooling maintainers
  - a "controlled hackathon" to measure impact (Nov 2026)
- Target: shift the developer growth rate by 30%+.
- **Relevance to us:** the official layer is being fixed right now. Our site should link to it and add what it lacks (see §4 and §5), not compete with it.

There is also a separate **OpenZeppelin proposal** for 11.79M ADA: an audited contracts library, reference implementations (liquid staking, self-repaying loans, tokenized MMF) and security capacity. It was discussed in Sept 2026; its outcome is **[UNVERIFIED]** [S34].

---

## 2. Root causes: why independent developers don't come

| # | Root cause | 2026 status | Evidence |
|---|---|---|---|
| 1 | **No economic pull: few users, little liquidity, and shrinking** | **STILL TRUE, and worse** | TVL $73M (#36), −87% since Dec 2024; tx/day down about 60% since Nov 2024; flagship apps closing [S20–S23][S27]. Developers follow capital: "Developers flow toward concentrations of capital, and that capital is pooling elsewhere" [S7]. |
| 2 | **Fragmented tooling, no "blessed path", hard zero-to-dApp** | **PARTLY FIXED; fix in progress** | #1 survey pain point [S9]. Portal 2026 added templates, a contracts library and AI skills [S12][S13]. But a Sept 2026 review still found "Start Here is effectively hidden", a Fundamentals 404, and deleted cardano-cli reference docs [S15]. `cardano-init` is not yet released. |
| 3 | **eUTxO mental model and concurrency design** | **STILL TRUE (inherent)** | The CF says the hard part is now the model, not the language: developers must master "the Extended UTXO (EUTXO) model" [S45]. Survey: "eUTxO is harder to grasp than account-based models"; "Need… clearer guidance for global state in eUTxO" [S9]. Concurrency workarounds (UTxO fragmentation, batchers) have been documented since 2023 [S46]. Nested transactions (Dijkstra) will help. |
| 4 | **Perception: "ghost chain", "Haskell-only", founder-centric drama, tribalism** | **Haskell myth OUTDATED; ghost-chain and drama perception STILL TRUE** | Aiken is at 75%+, so Haskell is optional [S8]. "Ghost chain" articles keep appearing in 2026 [S28][S47]. The treasury "civil war" and Hoskinson's break [S32][S29]. Forum thread title: "Cardano Has the Tech. So Why Is Everyone Else Winning Deals?" [S36]. |
| 5 | **Weak funding and talent pipeline (no Colosseum/Superteam equivalent)** | **IMPROVING but small-scale** | Survey: "Fear/hesitance from VCs and limited funding pathways beyond Catalyst" [S9]. 2026 additions: Orion fund (50M ADA first tranche) [S44], CF Venture Hub 2M ADA, the Cardano Accelerator Program (5 teams, CHF 10k each) [S48], a Catalyst Pilot (2–2.5M ADA, usage-based payouts) [S49], and Gimbalabs' Piece of Pie (33k ADA; 30+ builders → 15 mainnet projects → 3 with real users) [S50]. For comparison, Colosseum has processed **8,286** submissions across 5 hackathons [S51]. |
| 6 | **Opportunity discoverability** | **STILL TRUE** | A volunteer publishes "Cardano Weekly Opportunities" because announcements are "spread across different websites, forums, job boards, and social media platforms, making some opportunities easy to miss" [S35]. The talent pool is an email list only [S14]. At least three competing bounty boards are in early stages [S52]. |
| 7 | **No EVM compatibility** | **STILL TRUE** | Milkomeda C1 shut down in Sept 2025 [S41]. LayerZero (March 2026) brings messaging and liquidity, not Solidity execution [S31]. Solidity developers must rewrite their logic. |
| 8 | **Wallet / dApp UX** | **PARTLY TRUE** | Collateral confusion, and CIP-30 being "outgrown" [S53]. Lace 2.2 added Bitcoin support [S26]. Newcomers report "terminology overload" [S54]. |
| 9 | **Governance and funding friction for small builders** | **MIXED** | The treasury is huge, but proposals need a 67% DRep supermajority. 2026 budget winners were mostly established infrastructure teams (TxPipe ×5, Mithril, MLabs) [S55]. Hard for a solo developer to navigate. |
| 10 | **Infrastructure attrition** | **NEW in 2026** | TapTools APIs gone; Blockfrost's treasury vote debated [S36]. Builders worry the rug will be pulled on them. |

**Real voices (verbatim):**
- IO (Robertino Martinez), May 2026: *"Building is too hard, so most developers abandon or switch to a different blockchain."* [S10]
- CF survey respondents, 2025: *"Difficulty going from zero to a working dApp in any environment."* / *"Duplicated efforts and unawareness of existing work across teams."* / *"General chaos."* [S9]
- Developer-portal reviewer (GitHub issue #2008, 2026-09-11): *"No hosted reference documentation exists anywhere on the site"* for cardano-cli; content was deleted *"without advance notice to the contributing teams."* [S15]
- Wallet BD on the forum, July 2026: *"If Cardano users barely knew that a Hydra-powered trading platform existed, the issue was not necessarily the technology—it was the gap between building a product and getting it into users' hands."* [S36]
- Charles Hoskinson, June 2026: *"There's going to be a wave of failures in the ecosystem."* [S28]
- Andrew Westberg (2023 concurrency issue): fragmenting UTxOs is "complex, collision-prone, and poorly scalable" [S46].
- Forum newcomer, Aug 2026: friction comes from *"Terminology Overload: Essential concepts like Plutus, Mithril, Hydra, and Voltaire can feel intimidating"* [S54].

**Reddit:** Reddit's API and search blocked automated access, so I could not pull r/CardanoDevelopers quotes **[UNVERIFIED]**.

**Bottom line.** In 2026 the binding constraint is no longer "can I learn this?" Aiken, the portal and the templates have largely answered that. It is now **"why should I spend my scarce time here, and how do I get paid or funded?"** Our website can move that second question, which is a demand-and-incentive problem. Only the ecosystem can fix the first.

---

## 3. What worked elsewhere

### 3.1 Chain by chain

**Solana: the funnel from learning to earning to funding**
- **Solana Playground:** the official quick start runs entirely in the browser with no install. You still save a keypair and airdrop devnet SOL yourself [S56].
- **Anchor:** opinionated framework, used by 877 Colosseum projects [S51].
- **solana.com/developers:**
  - hero ("A manual for joining the Solana ecosystem. By builders for builders")
  - two CTAs, "Build Now" and "Get Support"
  - six learning programs, including RareSkills' **"Ethereum to Solana" course**
  - use-case tutorials (stablecoin payments, Token-2022, tokenization, games)
  - templates, changelog, newsletter [S57]
- **Colosseum:**
  - two fixed-date global online hackathons a year
  - an accelerator with up to **$250k pre-seed**
  - **Eternal**, a self-serve 4-week sprint you can start anytime
  - a cofounder directory
  - **Copilot**, an AI agent that checks your idea against 8,286 past submissions
  - Breakout drew 10k+ participants from 140+ countries and 1,412 final projects [S51][S58][S59]
- **Superteam:**
  - 25+ country chapters
  - **Earn** has 233k+ members and 2,730+ sponsor companies, offering bounties, projects and grants under one profile
  - Instagrants, plus local jobs and hackathons
  - members report earnings "3-5x" typical local salaries in emerging markets [S60][S61]
- **Lesson:** Solana did not win on documentation. It won by making the next step after learning **pay off immediately and publicly**, through bounties, prize money, pre-seed funding and local status.

**Ethereum: quests plus a scaffold plus hackathons**
- **ethereum.org/developers** puts three **Scaffold-ETH quests** up front (Tokenization → DEX → Stablecoin) and a one-liner `npx create-eth@latest`. It also offers LLM-friendly docs, Cyfrin video courses, and live hackathon and grant listings [S62].
- **Speedrun Ethereum** has 10 progressive challenges (now "AI-ready") plus a CTF. It links to **BuidlGuidl**, where completed challenges become a builder profile and a way into paid work [S63].
- **ETHGlobal:** 95+ events, 14,000+ projects, 150k+ community members, $13.5M+ in prizes and grants, $350M+ raised by alumni. Its 2025 events each drew 520–1,671 hackers [S64][S65].
- **Lesson:** a *visible, finishable* curriculum, a public portfolio, and recurring hackathons with sponsor prizes.

**Base: small, frequent, retroactive money**
- **Builder Rewards** (up to 2 ETH weekly) and **retro Builder Grants** (1–5 ETH, 20+ cohorts since March 2024).
- **Base Batches:** buildathon → incubator → Demo Day, $100k investment, up to $1M at demo day.
- **Base Ecosystem Fund** with Coinbase Ventures [S66][S67].
- The docs lead with use-case quickstarts and publish `llms.txt` [S68].
- **Lesson:** reward people who have already shipped, quickly and with little paperwork. It is cheap and builds trust.

**Sui and Aptos: AI-first onboarding (2026's new baseline)**
- **sui.io/developers** is now "Build on Sui with AI". It offers 33 agent skills, an MCP server with docs refreshed every few hours, a guided-scaffold agent, a starter template, three demo apps, and links to the DeepSurge hackathon and RFPs [S69]. Sui Overflow 2025 had 599 submissions from 85 countries and $550k in prizes [S70].
- **aptos.dev:** `create-aptos-dapp`, MCP, agent skills, a VS Code Move extension, keyless accounts, and grants and hackathons [S71].
- **Lesson:** in 2026, developers arrive *with a coding agent*. The first touchpoint should be "paste this into Claude/Cursor".

**NEAR:** the hero sentence states the unique edge ("…sign transactions on any chain from a single NEAR account"). It offers two quickstarts (`cargo near`, `create-near-app`), "Why NEAR?" with hard numbers (1.3s finality, $0.002 fees), and AI-agent tooling with `llms.txt` and MCP [S72].

**Polkadot: the closest analog, since its treasury also funds builders**
- **Fast-Grants** was a $500k treasury bounty paying grants of up to $10k, with approvals in as little as **72 hours**.
- Results: 119 applications, 28 approved, 19 funded and completed [S73].
- **Lesson:** a treasury *can* run a fast, small-ticket grants lane for new developers. Cardano's equivalents (Catalyst, budget proposals) are slow and heavy.

**Stellar (a non-EVM peer):** the SCF has funded 400+ projects through Kickstart bootcamps → Build Awards (up to $150k) → accelerator partners [S74].

**buildspace (cohort model, shut down 2024):** Nights & Weekends ran 6-week "build anything" cohorts with weekly lectures and an IRL demo day, and became "the largest accelerator in the world for any idea". It closed for founder reasons, not lack of demand [S75]. **Lesson:** time-boxed cohorts with social accountability convert. Gimbalabs' Piece of Pie is a Cardano-native version of this [S50].

### 3.2 Features that convert, compared

| Feature | Solana | Ethereum | Base | Sui/Aptos | NEAR | Cardano today |
|---|---|---|---|---|---|---|
| Zero-install first win (in-browser IDE) | ✅ Playground | ~ (Remix; Scaffold is local) | ✗ | ~ | ✗ | ~ Aiken Playground (contracts only); templates need Node, Blockfrost key, wallet and faucet [S13] |
| One-command scaffold | ✅ | ✅ `create-eth` | ✅ | ✅ `create-aptos-dapp` | ✅ | ~ `npx giget` templates; `cardano-init` pending [S10][S13] |
| "Coming from chain X" path | ✅ RareSkills ETH→SOL | n/a | n/a | ~ | ~ | ✅ portal "Cardano for Ethereum Developers" [S12] |
| Progressive quests with public portfolio | ~ | ✅ Speedrun/BuidlGuidl | ~ | ✗ | ✗ | ~ Asteria game, CTF, Andamio/PBL; no unified portfolio |
| Live bounty / opportunity board | ✅ Superteam Earn | ~ (Gitcoin, ETHGlobal) | ✅ Builder Rewards | ~ RFPs | ~ | ✗ (email talent pool, volunteer forum digest) [S14][S35] |
| Recurring global hackathon + accelerator | ✅ Colosseum | ✅ ETHGlobal | ✅ Base Batches | ✅ Overflow | ~ | ~ ad hoc (Buidler Fest is about 100 people; Piece of Pie; TOKEN2049 track) |
| Fast small grants / retro rewards | ✅ Instagrants | ~ | ✅ | ✅ | ~ | ✗ (Catalyst runs about one fund a year; pilot is usage-based) |
| AI-agent onboarding (skills/MCP/llms.txt) | ~ | ✅ | ✅ | ✅✅ | ✅ | ✅ Cardano Dev Skills, Mesh skills [S12][S18] |
| Local chapters | ✅ 25+ | ✅ | ~ | ~ | ~ | ~ (Nigeria, Indonesia, India, Vietnam hubs; uncoordinated) |
| Showcase gallery | ✅ | ✅ ETHGlobal showcase | ✅ | ✅ | ✅ | ~ Cardano Cube directory (650+), includes a project "Graveyard" [S40] |

---

## 4. Existing Cardano onboarding assets and their gaps

| Asset | What it is (2026) | Gap |
|---|---|---|
| **developers.cardano.org** (CF) | 431 pages: a 7-module curriculum, Builder Tools, **4 one-command templates** (Evolution+Vite, Mesh+Next.js, two **x402** starters), a **40-entry contracts library** across Aiken/Scalus and 6 off-chain SDKs, "Code with AI", "Agentic Commerce (x402)", "Cardano for Ethereum Developers", talent pool, weekly office hours, CTF, Asteria [S12][S13] | Navigation problems ("Start Here is effectively hidden"), deleted reference docs, and no notice to contributors [S15]. No in-browser end-to-end path. **No opportunity board, no rewards, no "why build here" economics.** Only an EVM persona so far; Solana and Web2 personas are planned by IO [S10]. |
| **Aiken** (aiken-lang.org) | Modern language, LSP, property testing, Playground, stdlib v3.1, PlutusV4 work underway. Used by Minswap, Sundae, CF, IO [S76] | Good for writing contracts. Off-chain and transaction-building remain the harder part [S9]. |
| **Mesh** (meshjs.dev) | TypeScript SDK (<60kB, 34k+ monthly downloads), React components, contract library, agent skills for 37+ tools, Hydra/Yaci/Midnight support [S77] | Competes with Evolution, Lucid Evolution, Blaze and others, which makes the "which SDK?" choice confusing [S9]. |
| **Gimbalabs / Andamio** | Plutus PBL (500+ students by its 4th iteration); Andamio LMS with on-chain credentials; Piece of Pie 12-week builder season [S78][S50] | Small scale; awareness is mostly inside the Cardano community. |
| **Cardano Academy (CF)** | Courses (e.g. a new digital identity course in Apr 2026; an Aiken eUTxO course) [S79] | Leans toward certificates rather than projects; no link to paid work. |
| **EMURGO Academy** | Paid developer training and corporate programs [S80] | Paid, and little evidence of 2025–26 activity **[UNVERIFIED]**. |
| **Intersect DevEx WG / Developer Advocates** (devex.intersectmbo.org) | Bi-weekly sessions (e.g. "Using AI in your Cardano dev workflow") and advocates [S16][S17] | Content is aimed at people already in the ecosystem. |
| **Cardano Cube** | Directory of 650+ projects, ecosystem map, governance explorer, Developer Hub, Graveyard [S40] | Built for project discovery, not developer conversion. |
| **Demeter.run (TxPipe)** | Hosted node, DB-Sync, Ogmios, Kupo and more [S81] | Not well connected to any learning flow. |
| **Buidler Fest (TxPipe)** | Toulouse 2024 → Da Nang 2025 → Buenos Aires (Mar 2026). Capped at about 100 people with a code-challenge requirement [S82][S9] | Deliberately small. It deepens existing developers but does not acquire new ones. |
| **Funding** | Catalyst F15 (18.5M ADA + 250k USDM; closed Jan 2026) [S83], Catalyst Pilot (Aug 2026) [S49], treasury budget (67% DRep threshold) [S55], Maintainer Retainer Program [S35], CF Venture Hub/CAP [S48], Orion fund [S44], Draper "UTXO Pitch Night" ($500k pool) [S35] | **Scattered across 10+ places with different cadences and rules.** No single navigator. |
| **Opportunity aggregation** | Volunteer "Cardano Weekly Opportunities" forum posts [S35]; Learn Cardano bounty preview; Cardano Bounties (Gimbalabs, waitlist); Githoney (TxPipe) [S52] | Unstructured, not filterable, no deadlines feed. The bounty platforms are early and competing. |

**What's missing overall:**
1. A single structured board of open opportunities.
2. An honest economic case for building on Cardano.
3. Rewarded, verifiable quests that lead into paid work.
4. Persona paths beyond EVM (Solana/Rust, Python, Web2, AI-agent builders).
5. An "open problems / ecosystem gaps" list, especially now that TapTools and JPG Store are gone and their niches are open.
6. A showcase of what *one or two people* have shipped.
7. Neutral tone. The independent ecosystem lacks a voice that admits weaknesses.

---

## 5. Recommendations for an independent builder-facing website

### 5.1 Positioning: Cardano's credible edges for builders in 2026

Only claim what holds up under a sceptical senior developer's scrutiny.

| Edge | Why it's credible | How to say it |
|---|---|---|
| **The treasury funds builders** | ~1.371B ADA (~$366M) on-chain treasury [S19]. Catalyst, budget withdrawals, Maintainer Retainers, Orion fund. Developers themselves rank "a treasury-funded ecosystem that empowers contributors" as a strength [S9]. | "The only top-tier chain where ADA holders vote to fund open-source builders, every year, on-chain." Show the live treasury balance and open calls. |
| **Deterministic transactions and fees** | eUTxO lets you know a transaction's outcome and fee before you submit it; a failed script costs only collateral. Developers name this as a top strength [S9]. | "No failed-tx gas burn. Simulate locally, know the exact fee." |
| **Native assets** | Tokens and NFTs are ledger-native, with no contract needed to mint or transfer [S12]. | "Mint a token without writing a contract." |
| **UTXO-native Bitcoin rails (BTCfi)** | Same UTXO mental model as Bitcoin; Cardinal (BTC↔Cardano) [S84]; Lace 2.2 signs Bitcoin transactions [S26]; Draper "UTXO Pitch Night" [S35]. | "If you think in UTXOs, you already think like Bitcoin. Build BTCfi where the model matches." Use carefully: many BTCfi products are still early **[UNVERIFIED maturity]**. |
| **Privacy via Midnight** | Mainnet launched March 2026; Compact (TypeScript-like) language; Catalyst Midnight category [S42][S83]. | "Selective-disclosure apps with a TS-like ZK language, settling next to Cardano." |
| **Agent payments (x402)** | Two official x402 starter templates; Masumi agent escrow [S13][S85]. | "Charge AI agents per request in tADA/USDM. Starter in one command." |
| **Formal methods and high assurance** | Plutus Core formalized in Lean 4; property testing in Aiken [S26][S76]. | Pitch this to finance and RWA builders, not hobbyists. |
| **Now-live institutional rails** | USDCx, LayerZero, Pyth, Dune [S31]. | "The 2024 objections (no stablecoin, no oracle, no bridge) are closed." |
| **Greenfield** | Low competition: whole categories are empty after JPG Store and TapTools closed. | "Be the default X on Cardano." State honestly that the user base is small. |

**Do NOT lead with:** "most GitHub commits" claims, TPS or Leios hype, "third-generation blockchain", or DeFi TVL. All of these backfire with experienced developers.

**Suggested hero line:** *"Get paid to build on Cardano. The on-chain treasury funds open-source builders. Here's every open opportunity, and the fastest path to your first shipped dApp."*

**Suggested sub-line:** *"Deterministic fees. Native assets. UTXO rails shared with Bitcoin. Privacy via Midnight. Small ecosystem, real funding: we'll show you both."*

### 5.2 Prioritized feature list

**P0: ship first (weeks 1–6)**

1. **"Should I build on Cardano?" honest page with live stats.**
   - Live data: TVL and rank, tx/day, stablecoin supply, treasury balance, open-funding total, number of active opportunities.
   - Pull from DefiLlama, Koios and Blockfrost, and stamp each number with the date it was fetched.
   - A **"Good fit / Bad fit"** table:
     - Good fit: grant-funded open source, payments/identity, agent commerce, BTCfi, privacy, RWA/enterprise.
     - Bad fit: anything that needs deep DeFi liquidity or a huge retail user base on day one.
   - Honesty is the differentiator: no official site will publish this, and it defuses "ghost chain" sniping.

2. **Live Opportunity Board.** This is the core, defensible asset.
   - Structured entries: type (grant / RFP / bounty / hackathon / job / accelerator / treasury-funded maintenance), amount, currency, deadline, eligibility, skill tags, persona, source URL, "last verified" date.
   - Sources to aggregate:
     - Catalyst funds and pilots
     - Intersect budget calls and the Maintainer Retainer Program
     - CF Venture Hub / CAP
     - the Orion fund
     - hackathon tracks (TOKEN2049, Piece of Pie, IO's Nov 2026 DevEx hackathon)
     - bounty platforms (Learn Cardano, Cardano Bounties, Githoney)
     - the Intersect job board and CF vacancies
   - Outputs: RSS/JSON feed, a weekly email, and `llms.txt` so agents can query it.
   - **Partner with, don't clone**, the "Cardano Weekly Opportunities" volunteer [S35] and the existing bounty boards. Offer them the structured database and the distribution.

3. **Persona router ("I'm coming from…").**
   - Personas: Solidity/EVM; Solana/Anchor (Rust); Web2 TypeScript; Python; Haskell/FP; AI-agent builder; Bitcoin developer.
   - Each persona page has three parts:
     - a **Rosetta-stone translation table**
     - one recommended starter (deep-linked to the official portal templates)
     - one first quest
   - Example Solidity → Cardano rows:

     | Solidity / EVM | Cardano |
     |---|---|
     | ERC-20 / ERC-721 | native assets + minting policy |
     | contract storage / mappings | datums on UTxOs; state-thread token pattern |
     | `msg.sender` | required signers / `extra_signatories` |
     | `require` reverts and burns gas | validator fails → tx rejected before fees (collateral only) |
     | proxies / upgradeability | parameterized validators, reference scripts |
     | events / logs | metadata + indexers (Oura/Kupo) |
     | Hardhat / Foundry | Aiken `check` + Yaci DevKit |
     | approve / transferFrom | not needed (spend authorization is per-UTxO) |
     | single shared pool state | batchers / UTxO sharding / (future) nested transactions |

   - Anchor developers should get a separate table. Solana's explicit account-passing model is conceptually closer to eUTxO than the EVM's, which is a strong onboarding angle.

4. **The 15-minute first win.**
   - One flow: embed or link the Aiken Playground for a validator → one-click Codespaces or devcontainer of the Mesh/Evolution template with preprod pre-configured → faucet link → "submit your first tx".
   - Switch to `cardano-init` when IO ships it [S10].

5. **AI-native from day one.**
   - Give every page an `llms.txt` and markdown export.
   - Add a "paste this into Claude Code/Cursor" block that installs Cardano Dev Skills and Mesh skills [S18].
   - Expose the opportunity board via a small MCP or JSON API. Sui shows this is now the baseline [S69].

**P1: months 2–4**

6. **Quests with on-chain verification** (Speedrun-style).
   - About 8 challenges: mint a native asset → vesting → escrow → multisig treasury → auction → oracle-consuming contract (Pyth) → x402 paywall → Midnight hello-world.
   - Completion is verified by checking a preprod transaction or script hash. No manual grading.
   - Completers get a public builder profile (CIP-68 credential or Andamio credential).
   - Top completers are routed to bounties and the talent pool.
   - Seek a small Catalyst or treasury grant to pay completion rewards, Base-style.

7. **"Ecosystem gaps / RFP" board.**
   - A curated list of products Cardano currently lacks that one or two developers could build. Seed it from the closures (analytics API after TapTools, NFT marketplace after JPG Store), survey asks (transaction-building abstractions, CBOR tooling, debugging), and Catalyst/OpenZeppelin gaps.
   - Each gap links to relevant funding. This answers "what should I build?", which is the second question after "why here?".

8. **Indie showcase.**
   - Projects shipped by teams of three or fewer: Piece of Pie yearbook, Catalyst-funded, hackathon winners.
   - Each entry shows team size, time to ship, funding received, and the stack used. This is social proof that a solo developer can make it.
   - Add a "lessons from the Graveyard" section that links to Cardano Cube's Graveyard [S40].

9. **Funding navigator.**
   - A decision tree: idea stage → Catalyst Concepts / bounties; working product → Catalyst Pilot (usage-based) or CAP; open-source infrastructure → treasury proposal or Maintainer Retainer; venture-scale → Orion / Draper.
   - Show the realistic timeline and approval threshold for each.

**P2: months 4+**

10. **Comparison tables** against Solana, Base, Sui and Ethereum L2s on fees, determinism, native assets, funding access, liquidity (honest) and tooling maturity, with sources.
11. **Builder Seasons with partners:** co-host quarterly 6–12-week cohorts with Gimbalabs and local hubs (Nigeria, Indonesia, India, Vietnam, Argentina). This copies the buildspace and Superteam chapter model.
12. **Monthly "State of Cardano for Builders" newsletter:** new opportunities, closed gaps, shipped projects, and honest metrics.

### 5.3 What NOT to do

- Don't duplicate the developer portal's docs. Deep-link to them and add the conversion layer.
- Don't launch yet another bounty platform with escrow. Aggregate the ones that exist. The ecosystem already has at least three competing ones [S52].
- Don't use tribal tone: no "Ethereum is centralized" jabs. Developers are multichain, and one in three works across several chains [S4].
- Don't publish anything without a "last verified" date. Stale opportunities destroy trust faster than having none.

### 5.4 Metrics to track (a funnel modeled on the IO initiative's KPIs [S10])

Visitors → persona page → first preprod tx (verified on-chain) → quest 3 completed → mainnet deploy → funding application or bounty claim → funded. Also track opportunity-board click-throughs per source, and email subscribers.

---

## Sources

- [S1] CF 2025 survey blog, citing Electric Capital 2024 (672 MAD / 276 FT / #15): https://cardanofoundation.org/blog/2025-developer-ecosystem-survey-results
- [S2] The Block on the EC 2024 report: https://www.theblock.co/post/330627/electric-capital-report-shows-solana-as-the-top-ecosystem-for-new-developers-in-2024 ; Hashlock per-chain MAD: https://hashlock.com/blog/blockchains-with-the-most-developers-in-2025 ; Aptos/Base/Sui 2024 notes: https://aelf.com/posts/electric-capital-developer-report-2024-key-takeaways-crypto-blockchain
- [S3] EC 2025 (Jan–Sep) new devs via EF: https://www.cryptoninjas.net/news/ethereum-dominates-2025-developer-landscape-with-over-16k-new-builders/ ; https://coinfomania.com/ethereum-tops-2025-dev-growth-solana-and-bitcoin-follow/
- [S4] Aggregate developer stats (EC/a16z/Artemis compilation): https://coinlaw.io/blockchain-developer-activity-statistics/
- [S5] CoinDesk, Artemis/EC commit collapse (2026-03-12): https://www.coindesk.com/tech/2026/03/12/crypto-developer-activity-sinks-to-multi-year-low-as-ai-absorbs-github-s-talent-boom
- [S6] Santiment Q1 2026 GitHub ranking: https://bex.co/blog/2026/03/15/santiment-q1-2026-github-activity-rankings-developer-commits-building-vs-marketing
- [S7] Motley Fool (Cryptometheus weekly devs): https://www.fool.com/investing/2025/06/08/3-warning-signs-that-its-time-to-sell-cardano
- [S8] cardano.org survey summary (2025-12-11): https://cardano.org/news/2025-12-11-developer-ecosystem-survey-2025/
- [S9] CF State of the Developer Ecosystem 2025 (full results; verbatim categories): https://cardano-foundation.github.io/state-of-the-developer-ecosystem/2025/ ; repo: https://github.com/cardano-foundation/state-of-the-developer-ecosystem
- [S10] IO Developer Experience Initiative (2026-05-18): https://www.iog.io/news/developer-experience-initiative
- [S11] IO DevEx project page and timeline: https://labs.iog.io/solutions/developer-experience
- [S12] Cardano Developer Portal: https://developers.cardano.org/ ; Cardano for Ethereum Developers: https://developers.cardano.org/docs/developers/cardano-for-ethereum-developers/ ; AI-assisted dev: https://developers.cardano.org/docs/developers/curriculum/start-building/ai-assisted-development/
- [S13] Portal templates and contracts library: https://developers.cardano.org/templates/ ; https://developers.cardano.org/templates/contracts/ ; https://developers.cardano.org/templates/x402-next/
- [S14] Talent pool: https://developers.cardano.org/talent/
- [S15] Portal 2026 review issue #2008: https://github.com/cardano-foundation/developer-portal/issues/2008 ; Ecosystem DevEx 2026 #1759: https://github.com/cardano-foundation/developer-portal/issues/1759 ; Portal 2026 #1758: https://github.com/cardano-foundation/developer-portal/issues/1758
- [S16] Intersect DevEx site: https://devex.intersectmbo.org/ ; Intersect DevEx survey: https://opensourcecommittee.docs.intersectmbo.org/about/open-source-office-oso/developer-advocate-program/state-of-developer-experience-survey
- [S17] Intersect survey forum post (2026): https://forum.cardano.org/t/state-of-developer-experience-survey/153216
- [S18] Cardano Dev Skills: https://www.claudepluginhub.com/plugins/cardano-foundation-cardano-dev-skills ; DevEx session 18: https://devex.intersectmbo.org/docs/working-group/q2-2026/sessions/18-cardano-ai-dev-workflow/session-resources
- [S19] Koios API (epoch_info, totals; pulled 2026-10-07): https://api.koios.rest/api/v1/epoch_info ; https://api.koios.rest/api/v1/totals
- [S20] DefiLlama chains API (pulled 2026-10-07): https://api.llama.fi/v2/chains ; https://defillama.com/chain/Cardano
- [S21] DefiLlama Cardano TVL history: https://api.llama.fi/v2/historicalChainTvl/Cardano
- [S22] DefiLlama stablecoins by chain: https://stablecoins.llama.fi/stablecoinchains
- [S23] DefiLlama Cardano DEX volume: https://api.llama.fi/overview/dexs/cardano
- [S24] Token Terminal weekly commits (CoinTurk): https://en.coin-turk.com/cardano-registered-233-code-commits-in-one-week-foundation-proposes-new-governance-forum/
- [S25] Chainspect developer activity: https://chainspect.app/dashboard/developer-activity
- [S26] Weekly development report 2026-07-31 (Lace 2.2 BTC, Plutus built-ins, Leios): https://cardano.org/news/2026-07-31-weekly-development-report/ ; 2026-04-10 (Lean 4 PlutusCoreBlaster): https://cardano.org/news/2026-04-10-weekly-development-report/
- [S27] The Block, TapTools wind-down: https://www.theblock.co/post/403457/taptools-winds-down
- [S28] MEXC explainer ("wave of failures", JPG Store, metrics): https://www.mexc.com/learn/article/is-cardano-ada-dead-its-founder-just-warned-of-a-wave-of-failures-/1
- [S29] The Defiant, TapTools: https://thedefiant.io/news/blockchains/cardano-s-taptools-winding-down-is-a-symptom-of-a-shrinking-chain
- [S30] Cointelegraph, TapTools: https://regional-front.cointelegraph.com/news/cardano-protocol-taptools-winds-down-after-five-execs-exit
- [S31] Critical Integrations closure report: https://cardano.org/news/2026-05-15-cardano-critical-integrations/ ; USDCx/LayerZero: https://www.kucoin.com/news/flash/cardano-launches-usdcx-midnight-and-first-onchain-audit-in-q1-2026
- [S32] TokenPost, May 2026 rift: https://www.tokenpost.com/news/business/20838
- [S33] IO research proposal standoff: https://www.tapbit.com/en/learn/article/cardano-ada-treasury-standoff-vision-2026-20260525
- [S34] OpenZeppelin Stack proposal (Governance Hour #10): https://forum.cardano.org/t/governance-hour-10-deep-dive-into-openzeppelin-stack-treasury-withdrawal-proposal-15-september/156932
- [S35] Cardano Opportunities This Week (Aug 31–Sep 5, 2026): https://forum.cardano.org/t/cardano-opportunities-this-week-august-31-to-september-5-2026/156653 ; Weekly Opportunities #4: https://forum.cardano.org/t/cardano-weekly-opportunities-4-september-20-2026/156901
- [S36] "Cardano Has the Tech. So Why Is Everyone Else Winning Deals?": https://forum.cardano.org/t/cardano-has-the-tech-so-why-is-everyone-else-winning-deals/155749
- [S37] Intersect 2026 budget status: https://x.com/IntersectMBO/status/2079914406712942863
- [S38] CoinGecko simple price API (pulled 2026-10-07): https://api.coingecko.com/api/v3/simple/price?ids=cardano&vs_currencies=usd
- [S39] Essential Cardano / weekly reports (project counts): https://cardano.org/news/2025-07-04-weekly-development-report/
- [S40] Cardano Cube: https://www.cardanocube.com/
- [S41] Milkomeda shutdown (forum thread quoting @Milkomeda_com): https://forum.cardano.org/t/current-status-of-cardano-evm-interoperability-milkomeda-rosen/153432
- [S42] Midnight mainnet guide: https://midnight.network/blog/getting-mainnet-ready-a-developer-s-guide ; https://cexplorer.io/article/cardano-s-privacy-partner-chain-midnight-to-launch-mainnet-this-march
- [S43] Van Rossem hard fork: https://cointelegraph.com/news/cardano-activates-van-rossem-hard-fork-paving-way-for-leios ; Dijkstra timeline: https://cardanoupgrades.docs.intersectmbo.org/general/hard-fork-working-group-meeting-minutes/1st-september-2026
- [S44] Orion Fund approval: https://cexplorer.io/article/cardano-community-approves-50m-ada-for-the-draper-dragon-orion-fund ; CF announcement: https://cardanofoundation.org/blog/orion-fund-initial-phase
- [S45] Building on Cardano without Haskell: https://cardano.org/news/2026-02-12-building-on-cardano-without-haskell/
- [S46] Concurrency issue #47 (Westberg): https://github.com/input-output-hk/Developer-Experience-working-group/issues/47
- [S47] Ghost-chain debate: https://memeburn.com/cardano-ghost-chain-debate-heats-up-as-devs-ship-record-code/
- [S48] CF Venture Hub / Cardano Accelerator Program: https://cardanofoundation.org/venture-hub/cardano-accelerator-program ; https://cardanofoundation.org/blog/venture-hub-expands-new-programs
- [S49] Catalyst Pilot: https://forum.cardano.org/t/its-open-apply-to-the-catalyst-pilot-or-join-decision-panel/156180 ; https://en.cryptonomist.ch/2026/08/10/cardano-ada-funding-round/
- [S50] Piece of Pie Builder Season results: https://forum.cardano.org/t/builder-season-an-experiment-in-long-form-hackathons-piece-of-pie-by-gimbalabs/155950
- [S51] Colosseum Copilot v2 (8,286 submissions; Anchor usage): https://solanacompass.com/news/colosseum-copilot-v2-can-now-review-your-solana-project-against-8286-hackathon-submissions
- [S52] Cardano Bounties: https://forum.cardano.org/t/introducing-cardano-bounties/154489 ; Githoney: https://lidonation.com/en/proposals/githoney-by-txpipe-good-first-issue-program-f13 ; DeTask: https://www.lidonation.com/en/proposals/detask-a-cardano-bounty-platform-for-developers-f13
- [S53] CPS-0010 wallet connectors: https://cips.cardano.org/cps/CPS-0010
- [S54] Newcomer analysis thread: https://forum.cardano.org/t/bridging-the-gap-a-newcomers-analysis-on-cardano-onboarding-growth/156420
- [S55] 2026 budget results: https://en.cryptonomist.ch/2026/06/21/cardano-2026-budget-results/
- [S56] Solana quick start (Playground): https://solana.com/docs/intro/quick-start
- [S57] Solana developers page: https://solana.com/developers
- [S58] Colosseum Eternal and schedule: https://blog.colosseum.com/announcing-colosseum-eternal-and-solanas-2025-hackathon-schedule/
- [S59] Breakout results: https://blog.colosseum.com/breakout-winners-confidential-spl-token-solana-ecosystem-report/
- [S60] Superteam Earn: https://superteam.fun/earn/
- [S61] Superteam: https://superteam.fun/
- [S62] ethereum.org developers: https://ethereum.org/en/developers/
- [S63] Speedrun Ethereum: https://speedrunethereum.com/
- [S64] ETHGlobal: https://ethglobal.com/
- [S65] ETHGlobal 2025 event stats (POAP feed): https://feed.poap.demo.goldsky.com/account/0x9666769ef08792a0f09e540f9cb16e60f583793b
- [S66] Base get funded: https://docs.base.org/get-started/get-funded
- [S67] Base Batches: https://base-batch-latam.devfolio.co/
- [S68] Base docs: https://docs.base.org/
- [S69] Sui developers: https://sui.io/developers
- [S70] Sui Overflow 2025 winners: https://blog.sui.io/2025-sui-overflow-hackathon-winners/
- [S71] Aptos developers: https://aptos.dev/
- [S72] NEAR docs: https://docs.near.org/
- [S73] Polkadot Fast-Grants final update: https://forum.polkadot.network/t/polkadot-fast-grants-programme-final-update-march-31-2026/17423
- [S74] Stellar Community Fund: https://stellar.org/blog/ecosystem/stellar-community-fund-4-0-bigger-better-faster-soroban ; https://thedefiant.io/education/tutorials/the-stellar-community-fund-evolves-to-bring-more-projects-to-mainnet
- [S75] buildspace shutdown: https://newsletter.failory.com/p/hidden-struggles
- [S76] Aiken: https://aiken-lang.org/ ; playground: https://play.aiken-lang.org/
- [S77] Mesh: https://meshjs.dev/
- [S78] Gimbalabs Plutus PBL: https://www.lidonation.com/proposals/translation-localization-and-implementation-of-the-gimbalabs-plutus-project-based-learning-program-f10
- [S79] CF April 2026 update (Academy): https://cardanofoundation.org/blog/april-2026-activities ; Aiken eUTxO course: https://cardanofoundation.org/en/academy/course/aiken-eutxo-smart-contracts-cardano
- [S80] EMURGO Academy: https://www.essentialcardano.io/article/emurgo-academy-is-official-sponsor-of-the-cardano-hackaton-in-argentina
- [S81] Demeter.run office hours: https://forum.cardano.org/t/developers-office-hours-76-demeter-run-infrastructure-made-simple-for-cardano-4-september/156560
- [S82] Buidler Fest #3: https://forum.cardano.org/t/digest-january-06-2026-buidler-fest-3-coming-to-buenos-aires-in-march-2026-cardano-enters-execution-phase-on-critical-integrations-python-sdk-for-cardanoscan-apis-spotlight-on-peter-bui-and-powered-by-cardano-cardano-foundation-q4-2025-report/152346
- [S83] Catalyst Fund 15: https://cexplorer.io/article/ada-holders-decide-the-next-wave-of-cardano-innovation-voting-open-january-13-27-2026 ; Midnight category: https://midnight.network/blog/how-to-get-involved-in-catalyst-fund15-s-midnight-category
- [S84] Cardinal BTC protocol: https://www.criptonoticias.com/tecnologia/cardano-primer-puente-interactuar-bitcoin/ ; Lace Bitcoin: https://lace.io/bitcoin
- [S85] Masumi / x402 on Cardano: https://www.dlnews.com/articles/defi/cardano-ada-founder-charles-hoskinson-praises-x402-integration/ ; https://docs.masumi.network/core-concepts/agent-to-agent-payments
