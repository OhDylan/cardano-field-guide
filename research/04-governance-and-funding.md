# 04 — On-chain Governance & Builder Funding on Cardano (state as of 2026-10-07)

> Audience: engineers coming from Ethereum/Solana or web2 who want to know (a) how Cardano's on-chain governance works and why it is programmable, and (b) every realistic way to get paid to build on Cardano in late 2026.
>
> Method: on-chain numbers come from the public Koios API (`api.koios.rest`), queried on 2026-10-07 at epoch 660. Everything else is cited inline as [S#] (see **Sources**). Anything I could not confirm is marked **[UNVERIFIED]**.

---

## TL;DR for builders

- **Cardano's treasury is a protocol-level account, and you can propose to spend it yourself.** Its balance is **~1,374.6M ADA** (epoch 660, Oct 2026). It was **~1,663.5M ADA** in April 2026 and **~1,681.7M ADA** in February 2025, so it has shrunk by roughly 289M ADA since spring 2026. The drawdown came mostly from Input Output's 2026 budget, the **120M ADA Cardano PRIME** program and the 2026 Intersect budget batch (Koios `/totals`).
- **Anyone can submit a treasury withdrawal.** It needs a refundable **100,000 ADA deposit**, **≥67% of active DRep stake**, and a **Constitutional Committee** "constitutional" vote. SPOs do not vote on withdrawals. A typical proposal is live for about **6 epochs (~30 days)**.
- **Spending is capped by a "Net Change Limit" (NCL).** The current NCL is **500M ADA for epochs 613–713 (13 Feb 2026 → ~3 Jul 2027)**. DReps raised it from 350M in July 2026 [S1, Koios]. About **457M ADA** of 2026 withdrawals have been enacted, so roughly **43M ADA of headroom** is left until July 2027. That is my own calculation from on-chain data. Big new asks will struggle until a new NCL is agreed.
- **Project Catalyst is now run by the Cardano Foundation, and it has changed shape.** Fund 15 (18.5M ADA + 250K USDM) was paused in January 2026. That money, plus Fund 16's, went back to the treasury, and stewardship moved from IOG to the CF [S2–S5]. Its replacement is the **Catalyst Pilot 2026**: 2.5M ADA, grants of 50k–200k ADA, 12 teams picked on 24 Sep 2026, with applications closed 20 Aug 2026 [S6–S8].
- **Cardano PRIME is a 120M ADA DeFi-growth program, not a Catalyst fund.** It was proposed by AlphaGrowth. It was submitted on 9 Jul 2026, approved with ~73% DRep support, and enacted on **17 Aug 2026** [S9–S12]. It carries **$5.6M (₳35M) of ecosystem grants**, and an RFP opened in September 2026 for wallets, an EVM↔Cardano bridge, lending/vault standards and liquidity tooling [S13, S14]. **No public deadline found [UNVERIFIED].**
- **FC Barcelona's "Barça Fan Lab" was funded through Catalyst Fund 13.** The Catalyst proposal was submitted with **Andamio** as tech partner and requested **1,425,000 ADA**. As of launch on **25 Sep 2026**, 789,500 ADA had been paid and 635,500 ADA remained, with 2 of 6 milestones complete [S15–S18].
- **Node diversity is real and treasury-funded.** Amaru (Rust, PRAGMA) received 10.1M ADA in 2026 and targets mainnet block production in **Nov 2026**. Dingo (Go, Blink Labs) received 6.9M ADA. HLabs funds Pebble and TypeScript tooling (4.6M ADA). IOG **stopped Acropolis** in April 2026 and returned the money [S19–S23].

---

## 1. Governance (Voltaire / CIP-1694)

### 1.1 Timeline

| Date | Event | Source |
|---|---|---|
| Sep 2024 | **Chang hard fork**: CIP-1694 governance goes live under an interim constitution and Interim Constitutional Committee (ICC) | [S24] |
| Jan 2025 | **Plomin hard fork** (protocol v10) gives DReps full voting power (treasury withdrawals, constitution updates) | Koios; [S24] |
| 30 Jan 2025 | First Cardano Constitution submitted on-chain. It was drafted at a constitutional convention (Buenos Aires + Nairobi, Dec 2024) | [S24] |
| Epoch 541 → effective **23 Feb 2025** | Constitution **ratified** with ~85% DRep approval and ICC unanimity | [S24, S25] |
| Jul–Aug 2025 | **2025 budget**: 39 Intersect-administered withdrawals; ₳264M approved; first use of on-chain treasury smart contracts | [S26, S27] |
| Jul 2025 (epoch 581) | Elected Constitutional Committee replaces the ICC | Koios |
| Dec 2025 | **₳70M Cardano Critical Integrations** budget (USDCx, LayerZero, Pyth, Dune, custody) enacted | Koios; [S28] |
| Jan 2026 (epoch 609) | **Constitution v2.4** enacted (~79% DRep). "Budget Info Actions" are no longer recognised; each withdrawal must carry its own full justification | Koios; [S29] |
| Feb–Mar 2026 | Catalyst paused and stewardship moves to the CF (info action, 83% yes) | Koios; [S2–S4] |
| Apr–Jun 2026 | IO's 2026 budget: 6 of 9 proposals pass. Intersect's 2026 budget process runs | Koios; [S30, S31] |
| May 2026 | Cardano Summit 2026 funding fails at **65.06% vs 67%** needed, and the summit is cancelled | Koios; [S32] |
| Jul 2026 (epoch 644) | **van Rossem hard fork** (protocol v11) enacted | Koios |
| Jul 2026 | NCL raised to **500M ADA** for epochs 613–713. Constitutional Committee 2026 update enacted (epoch 654) | Koios |
| 17 Aug 2026 | **Cardano PRIME (₳120M)** enacted | [S10] |
| Oct 2026 (live) | OpenZeppelin Stack (₳11.79M) is voting, currently ~3.8% yes / 96% no. A minPoolCost 170→75 ADA parameter change and an SPO poll on k=500→1000 are also open | Koios |

### 1.2 The three bodies

| Body | Who | What they vote on | Current numbers (epoch 660) |
|---|---|---|---|
| **DReps** (Delegated Representatives) | Anyone can register with a **500 ADA** deposit. ADA holders delegate voting power, which is liquid and revocable | Every action type except parameter changes in the "security" group, which only SPOs vote on. **Sole decision-makers (with the CC) for treasury withdrawals** | **858** DReps. ~15.8B ADA delegated to DRep options, of which ~**10.4B ADA** sits with the predefined **"Always Abstain"** option (mostly exchanges and custodians). DReps go inactive after **20 epochs** without voting |
| **SPOs** (Stake Pool Operators) | ~3,000 pools, weighted by stake [UNVERIFIED count] | No-confidence, committee changes, hard forks, security-relevant parameters, info actions | Thresholds of 51% (see below) |
| **Constitutional Committee (CC)** | Elected members (7 seated now, minimum size 5 since Jun 2026). Members can be script-based (multisig) credentials | Checks whether each action is **constitutional**. Does not judge whether it is a good idea | Quorum **2/3**. Max term 146 epochs. A 2026 election via Intersect's Hydra-based voting replaced 4 seats; KtorZ and Cardano Japan Council finished their terms [S33, S34] |

### 1.3 Governance action types and thresholds (live protocol parameters, epoch 660)

| # | Action | DRep threshold | SPO threshold | CC needed? |
|---|---|---|---|---|
| 1 | Motion of no-confidence | 67% | 51% | No |
| 2 | Update committee / threshold (normal state) | 67% | 51% | No |
| 2b | Update committee (no-confidence state) | 60% | 51% | No |
| 3 | New Constitution / guardrails script | **75%** | — | Yes |
| 4 | Hard-fork initiation | 60% | 51% | Yes |
| 5 | Protocol parameter change: network / economic / technical groups | 67% | — (51% only for security-group params) | Yes |
| 5b | Protocol parameter change: governance group | **75%** | — | Yes |
| 6 | **Treasury withdrawal** | **67%** | — | Yes |
| 7 | Info action (signalling only, never enacted) | Constitution uses >50% of active stake for things like NCL agreement | — | — |

Other live parameters: **governance action deposit = 100,000 ADA** (refunded to the return address once the action is enacted, expires or is dropped), **gov_action_lifetime = 6 epochs (~30 days)**, **DRep deposit = 500 ADA**, **treasury cut (τ) = 20%** of each epoch's rewards pot, **monetary expansion (ρ) = 0.3%** of reserves per epoch (Koios `epoch_params`).

### 1.4 The treasury in numbers

| Metric | Value | Source |
|---|---|---|
| Treasury balance (epoch 660, ~6 Oct 2026) | **1,374,649,329 ADA** | Koios `/totals` |
| Treasury (epoch 633, ~late May 2026) | 1,643,810,583 ADA | Koios |
| Treasury (epoch 623, ~Apr 2026) | 1,663,540,425 ADA | Koios |
| Treasury (epoch 541, Feb 2025, when the Constitution was ratified) | 1,681,719,994 ADA | Koios |
| Gross inflow (recent epochs) | ~3.6–3.7M ADA per epoch ≈ **~265M ADA/yr** (my estimate from epochs 658→660) | Koios |
| Reserves left to emit | ~6.08B ADA | Koios |
| Enacted withdrawals from proposals submitted in 2025 | **40 actions, ~347.0M ADA** | Koios `/proposal_list` |
| Enacted withdrawals from proposals submitted in 2026 (so far) | **30 actions, ~457.4M ADA** | Koios |
| Withdrawal proposals submitted in 2026 that **failed or expired** | 28 of 59 | Koios |
| Current NCL | **500M ADA, epochs 613–713** (agreed Jul 2026, 62% yes; replaced the 350M NCL agreed Jan 2026) | Koios; [S1] |
| Paid out over the 90 days to Aug 2026 | 269.6M ADA | [S35] |

**Takeaway for builders:** at today's ~$0.21/ADA [S36], the remaining ~43M ADA of 2026–27 NCL headroom is only about $9M. The next NCL period, or a new NCL info action, is the gating event for any big treasury ask.

### 1.5 Budget cycles

**2025 budget (the first one).** Intersect ran an off-chain pipeline: 194 proposals → Ekklesia poll → 40 cleared → 39 named Intersect as administrator → on-chain withdrawals. DReps approved about **₳264M** (19 Aug 2025) [S26, S27]. The largest enacted items were:

| Item | ADA |
|---|---|
| IOE Core Development (Input Output) | 96,817,080 |
| Catalyst 2025 (IO). F15/F16 portions were later returned to the treasury | 69,459,000 |
| IO Research "Cardano Vision" | 26,840,000 |
| Intersect MBO operations | 15,750,000 |
| Cardano Builder DAO | 12,000,000 |
| Tweag core projects | 11,070,323 |
| Ecosystem events / marketing (two items) | 6,000,000 each |
| OSC Paid Open Source Model | 5,885,000 |
| Stablecoin / native asset support | 4,000,000 |
| Midgard (optimistic rollups) | 2,162,096 |
| Long tail of dev tooling: Blockfrost 1.3M, zkFold 1.16M, Scalus 658K, Gerolamo 579K, Eternl 583K, PyCardano 315K, OpShin 200K, Pallas/Dolos/UTxO RPC 221K each, Lucid Evolution 131K, BloxBean 100K, MLabs Plutarch 243K… | — |

Separately, a **₳70M Critical Integrations Budget** (Pentad) was enacted in Dec 2025 (Koios).

**2026 budget.** The framework info action got 68.4% yes (Mar 2026) [Koios, S37]. The process [S38] ran as follows:

- Submissions on Ekklesia: **16 Apr – 8 May 2026**. Minimum proposal size **100,000 ADA**, submission fee **1,000 ADA** [S37].
- Refinement: 8–22 May.
- **Off-chain DRep "Hydra-voting"** with a **67% bar**: 26 May – 12 Jun. Results on 19 Jun.
- On-chain withdrawals submitted after 29 Jun. Intersect only submits for vendors who chose Intersect as administrator.

The resulting batch (23 Jun 2026, all enacted) included:

| Recipient | ADA |
|---|---|
| Intersect governance coordination | 25.4M |
| Mithril | 3.81M |
| Tx3 by TxPipe | 1.68M |
| Hardware wallets | 1.31M |
| TSC support | 1.19M |
| MLabs Plutarch | 1.16M |
| Pallas, Oura, Dolos, UTxO RPC (TxPipe) | 540,750 each |

Rejected or expired from that batch: Wirex payments, Builder DAO (20M), an enterprise ticketing platform, Blockfrost non-profit (9.8M) and more (Koios).

**Input Output 2026.** Nine proposals totalling about $38.9–46.8M, voted 23 Apr – 24 May 2026, roughly half of the 2025 ask [S30, S31, S39].

- **Passed:** Consensus/Leios (27.7M ADA, ~84% yes), Cardano Maintenance (with Ensurable Systems, 62.1M), Cardano Upgrades (13.1M: microfees CIP-159, multi-asset treasury CPS-23, Babel fees), High Assurance (13.1M), Plutus (with VacuumLabs, 11.9M), Developer Experience (3.6M). Also IO Research "Vision 2026" (32.9M) and IO Hydra (5.1M) (Koios).
- **Failed:** **Pogun** (BTC DeFi; 12.29M; ~35.7% yes), IO & Midgard L2 Scalability (10.4M) (Koios; [S40]).

### 1.6 Notable passes and failures

| Action | Result | Why it's interesting |
|---|---|---|
| Cardano PRIME ₳120M (AlphaGrowth) | **Enacted** 17 Aug 2026 (~73% of participating DRep stake, CC 6/7) | The largest single growth program. Phase-gated, with a 5-member Operating Group that can withhold ~₳90M [S9–S12] |
| Draper Dragon **Orion Fund** ₳50M | **Enacted** (~75% yes, Mar 2026) | The first treasury-backed **VC fund** that takes equity/token positions, with returns flowing back to the treasury. Target fund size $80M [S41] |
| Critical Integrations V1 ₳70M / V2 ₳23M | Enacted | USDCx, LayerZero, Pyth, Dune, Fireblocks. These are directly useful to dapp builders [S28] |
| CF as Catalyst managing entity (info) | 83% yes | Catalyst moved from IOG to the CF |
| Cardano Summit 2026 (₳14.1M, revised to ₳7.8M) | **Failed** (65.06% vs 67%) | Shows how hard the 67% bar is, even with a CC "yes" and a headcount majority [S32] |
| Pogun (IO) | Failed (~36%) | DReps pushed back on founding-entity product bets [S40] |
| Reduce CC minimum size 7→5 | Failed in Feb, **passed in Jun 2026** (80%) | Iteration through governance |
| "Reforming Treasury Governance" (info) | Failed (~7%) | — |
| OpenZeppelin Stack ₳11.79M | **Live, failing** (~3.8% yes as of epoch 660, expires epoch 661) | Shows the DRep bar for outside vendors |

### 1.7 Why devs should care: governance is programmable

- **Treasury smart contracts (Aiken).** The 2025 budget was the first time treasury money flowed into audited on-chain contracts instead of a bank account. Sundae Labs and Xerberus built `treasury.ak` and `vendor.ak` [S42, S43]. Funds can't be staked or used to vote; they are auto-delegated to the "always abstain" DRep, as the constitution requires. Expired funds can be swept back to the treasury by **anyone**. **Milestones vest per vendor**, and there are 7 permissioned actions (reorganize, sweep-early, disburse, fund, pause, resume, modify). Each permission is a native-script-like condition plus optional **withdrawal-script hooks for custom logic**. The contracts were audited by TxPipe and MLabs. There is a TypeScript SDK (`@sundaeswap/treasury-funds` on npm), a CLI, and an on-chain metadata standard so indexers can follow the money. Repo: `github.com/SundaeSwap-finance/treasury-contracts`.
- **CIP-108 / CIP-100 metadata.** Every action carries anchor-hashed JSON-LD metadata (title, abstract, motivation, rationale). It is fully indexable: this report's numbers came straight from Koios `/proposal_list` and `/proposal_voting_summary`.
- **Script-based governance credentials.** DReps and CC members can be **Plutus or native scripts**, so DAOs, multisigs and token-weighted committees can vote as one DRep.
- **Guardrails script.** A Plutus script attached to the Constitution automatically rejects out-of-bounds parameter changes and non-ADA withdrawals (TREASURY-03a) [S29].
- **Tooling map** [S44, S45, S34]:

| Tool | What it does |
|---|---|
| **GovTool** (gov.tools; Intersect; now maintained by **Sireto**) | Register as a DRep, delegate, propose, vote |
| **1694.io** | DRep profiles and metadata |
| **1694.tools** | Proposal explorer |
| **Tempo.vote** | Governance UI (received a 2025 maintenance budget request) |
| **Cardanoscan / Cexplorer / AdaStat / CardanoCube** | Explorers with governance views |
| **Ekklesia + Intersect Hydra-voting** | Off-chain budget polls and CC elections |
| **Cardano Ballot** (Cardano Foundation) | Community polls [UNVERIFIED current use] |
| **Koios / Blockfrost / UTxO RPC** | APIs for governance data |

### 1.8 How to get money directly from the treasury (step by step)

1. **Pre-socialise.** Post on the Cardano Forum and in DRep channels (Intersect's DRep forum and X spaces). Pitch to large DReps; the Cardano Foundation publishes its own review criteria [S46]. Expect 1–3 months.
2. **Check the NCL.** Under TREASURY-02a, a withdrawal that would exceed the current NCL is unconstitutional. About 43M ADA remains until July 2027.
3. **Choose a route:**
   - (a) **Intersect budget process**, which runs annually with an Ekklesia window around April. Minimum 100k ADA, 1k ADA fee, and a 67% off-chain DRep vote. Intersect then submits the withdrawal and acts as administrator using the treasury contracts.
   - (b) **Self-submit** with your own administrator, as Amaru (PRAGMA), Dingo (Blink Labs) and PRIME (Intersect as constitutional administrator) did.
4. **Meet Constitution v2.4, Article II §7** [S29]:
   - State the purpose, delivery period, costs and refund conditions.
   - Disclose any treasury funding received in the last 24 months.
   - **Budget for an independent audit** and oversight metrics.
   - Name **one or more administrators**.
   - Keep funds in segregated, auditable accounts delegated to always-abstain.
   - Use immutable (IPFS) metadata links.
5. **Submit on-chain.** Lock the **100,000 ADA deposit**, which is refunded. Voting runs about 6 epochs (~30 days).
6. **Win ≥67% of active DRep stake plus a CC "constitutional" vote.** Then the withdrawal is enacted at the next epoch boundary.
7. **Deliver against milestones.** The administrator releases tranches from the vendor contract and can pause disputed milestones.

Realistic size: hundreds of thousands to tens of millions of ADA. Track record in 2026: **31 of 59 withdrawal proposals passed**. Small, unknown teams rarely clear 67%; established infrastructure maintainers usually do.

---

## 2. Project Catalyst

| Fund | Launched | Budget | Funded / submitted | Notes |
|---|---|---|---|---|
| F13 | Sep 2024 | ₳46.5M (per site) | 199 / 1,640 | Categories included Developer support (₳8M), Ecosystem growth (₳5M), Enterprise R&D (₳15M) [S2] |
| F14 | Jun–Jul 2025 | ₳18.6M | 131 / 1,280; results 14 Oct 2025 | [S2, S47] |
| F15 | Nov 2025 | ₳18.5M + 250K USDM; "multi-chain" (Midnight "Compact DApps" category) | 761 proposals; **voting paused 13 Jan 2026; never ran** | Funds returned to the treasury [S3–S5, S48] |
| F16 | — | — | **Cancelled in proposed form** | [S5] |
| **Catalyst Pilot 2026** | Applications open Aug 2026, closed **20 Aug 06:00 UTC** | **₳2.5M** (first announced as ₳2M) from undistributed ADA of earlier funds, **not new treasury money** | 117 applications → 31 finalists → **12 selected (24 Sep 2026)** | Grants of 50k–200k ADA. Paid 40% upfront, up to 40% on usage KPIs (measured via network fees and tx labels), 20% for sustained usage, and up to a 50% bonus for the top 3. Must ship to mainnet within 3 months. Themes: oracles (Pyth), stablecoins (USDCx), programmable tokens, on-chain identity. Picked by a 35-person panel (28 community + 7 CF) instead of public voting [S6–S8] |

**Lifetime stats:** 2,221 proposals funded, $150M+ distributed, 3.26M votes cast, and 1,500 completed projects as of Dec 2025 [S2]. cardano.org claims "$200M+ across 2,200+ projects" for all programs [S49]. More than 500 projects are still being paid against milestones (F10–F14) [S4].

**The 12 Pilot 2026 teams** [S7]: anyqr (SyncAI, USDCx QR payments), AnyToAny (VIA Labs bridging), ATLAS (perps), CIP-0170 Identity Rails (Aline Health), Compliant Secondary Trading (Anvil), In-Place Collateral Swaps (Dano Finance), L4VA (RWA vaults), Liqwid Pyth integration, nuhuman (Nucast traceability), OriLife (GreenSun), Pay (Cexplorer invoicing), Regulated BRL Settlement (PagFinance).

**How to apply now:** watch `projectcatalyst.io/apply`. The CF says a longer-term Catalyst strategy will follow the pilot; no next fund date has been announced **[UNVERIFIED]**.

### 2.1 FC Barcelona × Cardano: exact facts

| Fact | Detail |
|---|---|
| What | **Barça Fan Lab**, a digital fan-participation platform. Fans sign in with **BarçaID**, which automatically creates a personal **Cardano wallet/portfolio**. They complete learning pathways and community activities and earn **verifiable credentials and access tokens**. Four content areas: club history and identity; sustainability, inclusion and women's empowerment; physical-to-digital fan participation; practical Web3 (collectibles, credentials, loyalty, NFTs, smart contracts). Loyalty and collectibles are on the roadmap [S15–S18] |
| Who built it | **Andamio**, the tech partner, with its on-chain credential and learning platform. Part of the club's **"Barça Vision"** Web3 initiative [S16, S50] |
| Funding | **Project Catalyst Fund 13**, proposal "FC Barcelona – Fan engagement infrastructure (Cardano)". **Requested 1,425,000 ADA. Distributed 789,500 ADA; 635,500 ADA remaining. 2 of 6 milestones done, 1 in progress** (as reported 25 Sep 2026) [S17, S51]. Exact F13 category: [UNVERIFIED] |
| Dates | F13 voting/results were in late 2024 [UNVERIFIED exact date]. **Public launch 25 Sep 2026** (FC Barcelona press release; coverage 25–28 Sep 2026) [S15–S18] |
| What it is *not* | Not a Cardano Foundation or IOG sponsorship deal, and not a treasury withdrawal. It is a community-voted Catalyst grant with milestone payouts |

---

## 3. Other funding and support (status Oct 2026)

| Program | Run by | Type / size | Status / deadline | Source |
|---|---|---|---|---|
| **Cardano PRIME** ecosystem grants | AlphaGrowth (exec.), Intersect (constitutional administrator), 5-member Operating Group (Blink Labs, CoinseLion, Midgard Labs, Input Output, Tweag) | ₳35M ($5.6M) grants + ₳27M LP incentives. A single grant above $1M must go through a separate on-chain action. "$1 of grant should support $50 of sustained TVL" | **RFP open since ~Sep 2026**. Target areas: dual-stack wallets, EVM→Cardano bridge, lending/vault standards, liquidity tooling. Entry point: alphagrowth.typeform.com/cardano-prime. **Deadline: [UNVERIFIED]**. The Phase 3 gate is at month 4 (~mid-Dec 2026) | [S9, S12–S14] |
| **Catalyst Pilot 2026** | Cardano Foundation | 50k–200k ADA | Closed 20 Aug 2026. 12 teams selected | [S6–S8] |
| **Treasury withdrawal** (self or Intersect) | DReps + CC | 100k ADA min (Intersect process) up to tens of millions | Always open, but the 2026 Intersect window has closed. Next annual process: [UNVERIFIED, likely ~Q1–Q2 2027]. NCL headroom ~43M ADA | §1.8 |
| **Orion Fund** (Draper Dragon, CF admin support, Draper University) | VC | $80M target (₳50M tranche 1 from the treasury). Acceleration through Series A; RWA and institutional DeFi focus | Open (rolling) | [S41, S49] |
| **Cardano Builder DAO** | Member DAO (NMKR, ADA Handle, Minswap, Indigo, Iagon, TapTools, Vespr…) | 12M ADA in 2025, for live apps driving MAU/TVL. **2026 request (20M) failed** | Membership-based [UNVERIFIED how to join] | [S52], Koios |
| **Cardano Accelerator Program (CAP)** / Venture Hub 2.0 | Cardano Foundation | Up to **2M ADA** committed for 2026. Fall '26 cohort: 5 teams, CHF 10k each, 10 weeks, Demo Day. Theme "Real-World Trust" (digital product passports, identity, traceability) | Fall '26 closed 5 Jun 2026; program started end of Sep 2026 | [S53, S54] |
| **Genesis Pre-Accelerator / Apex Growth Accelerator** | Draper University (listed on cardano.org) | Up to $20k (4 weeks) / up to $70k for ~3.5% equity (10 weeks) | Recurring | [S49] |
| **SDG Blockchain Accelerator** | UNDP | Public-sector pilots | Recurring | [S49] |
| **Maintainer Retainer Program** | Intersect OSC/TSC (Paid Open Source Model, ₳5.9M in 2025) | Recurring pay for core and community maintainers (cardano-node, db-sync, ledger, Hydra, Oura…). 30-day evaluation | Open | [S55, S56] |
| **Intersect committees** (OSC, TSC, Product, Budget, Civics, MCC) | Intersect | Paid committee and working-group seats. 2026 elections: 102 candidates for 37 seats | Annual elections in April | [S57] |
| **Developer Experience Working Group** | Intersect OSC | 12-session Q1 2026 program, workshops, open clinics. Developer Advocate program | Ongoing | [S58] |
| **Midnight**: Night Sky Accelerator, Build Club, hackathons | Midnight Foundation / Shielded Technologies | 10-week accelerator from Jul 2026. Hack Buenos Aires (7–8 Aug 2026, $10k). MLH Midnight hackathons (May 2026 and others). Buildathon ($12.5k grants) | Rolling | [S59–S61] |
| **Hackathons** | Various | Cardano Summit 2025 "Layer Up" (134 devs, up to $30k). Charli3 Oracles hackathon (Apr 2026, 40k ADA, CF + Catalyst funded). Gimbalabs **Piece of Pie** (12-week build-in-public, to 19 Jul 2026) | Periodic | [S62–S64] |
| **TOKEN2049 Singapore Cardano booth** | Cardano Foundation (₳3.3M treasury-funded platinum sponsorship) | 20 projects; flights up to $2k plus 3 nights covered | Applications closed 25 Aug. **Event is today–tomorrow (7–8 Oct 2026)** | [S65], Koios |
| **EMURGO** Ventures / Africa / Academy | EMURGO | Historic $100M vehicle (2021) [S66]; Academy courses | **Caution:** EMURGO **left the Pentad on 8–9 Jul 2026** after the SecondFi (formerly Yoroi) wallet exploit (~16.1M ADA from 374 wallets). SecondFi/Yoroi is shutting down. Current Ventures activity [UNVERIFIED] | [S67, S68] |

**Events.** Buidler Fest #2 was held in Da Nang (24–25 Apr 2025) and #3 in **Buenos Aires (24–25 Mar 2026)** [S69]. **Cardano Summit 2025** was in Berlin (11–13 Nov 2025; 1,460+ in person, 26k online) [S62]. **Cardano Summit 2026 was cancelled** after its treasury vote failed [S32]. Rare Evo 2026 was in Las Vegas, 28–31 Jul [S65]. A Buidler Fest #4 date is [UNVERIFIED].

**Community and dev surfaces:**

- **developers.cardano.org**. Its funding page now redirects to cardano.org/grants-funding, which lists 10 programs [S49].
- **CIPs** at cips.cardano.org. To propose one, open a PR to `cardano-foundation/CIPs` with "?" as the number. Editors triage in bi-weekly public Discord meetings. Statuses are Proposed → Active → Inactive. Use a **CPS** (CIP-9999) to describe a problem before proposing a solution [S70].
- Also: Cardano Forum, Cardano StackExchange, IOG Technical Discord, Intersect Discord, and Aiken / TxPipe / Blink Labs Discords.

---

## 4. Ecosystem organisations map

| Org | What it does | Signals for third-party devs |
|---|---|---|
| **Input Output (IO / IOG)** | Original core engineering and research: Haskell node, Plutus, Leios (mainnet target end-2026), Hydra, Mithril, Midnight R&D. Received ~₳131M across 6 approved 2026 proposals plus 32.9M for research. Halted **Acropolis** (modular Rust node) and Tiered Pricing in Apr 2026 and returned 4.1M ADA; the Acropolis code stays open source [S21, S30] | Developer Experience Initiative (3.6M ADA) aims for "+30% dev growth". IO cites ~550 active Cardano devs (Koios metadata) |
| **Cardano Foundation (CF)** | Swiss foundation. Now **runs Project Catalyst**, the Venture Hub/CAP, cardano.org and the developer portal, Cardano Ballot, events. PRAGMA founding member. Large DRep voter | Catalyst Pilot, CAP, TOKEN2049 booth |
| **EMURGO** | Commercial arm: ventures, Academy, Africa. **Stepped down from the Pentad Jul 2026** after the SecondFi exploit | Treat as reduced presence [S67] |
| **Intersect (MBO)** | Member-based org: committees (TSC, OSC, Budget, Product, Civics), **budget administrator** for most treasury money, Hydra-voting, GovTool stewardship (now Sireto), Constitutional Amendment Portal. Received 25.4M ADA for 2026 | Membership is open. Committees pay. Maintainer Retainer |
| **Midnight Foundation / Shielded Technologies** | Midnight privacy chain (ZK, **Compact** language, NIGHT token TGE Dec 2025, **mainnet genesis 17 Mar 2026**). Both founded May 2025 [S59, S71] | Accelerators, hackathons, Catalyst F15 "Compact DApps" track (paused) |
| **Pentad** | IO, CF, EMURGO (exited Jul 2026), Intersect, Midnight Foundation. Coordinates **Critical Integrations** (USDCx, LayerZero, Pyth, Dune, Fireblocks) and "Pentad V2" [S28] | — |
| **PRAGMA** | Open-source member association (Apr 2024): CF, Blink Labs, dcSpark, Sundae Labs, TxPipe. Hosts **Aiken** (smart contract language) and **Amaru** (Rust node) [S19] | Contribute to Aiken/Amaru. The 2026 maintainer committee includes Matthias Benkort (KtorZ) and Pi Lanningham |
| **TxPipe** | Rust infrastructure: Pallas, Dolos (data node), Oura, UTxO RPC, **Tx3** (tx DSL), Demeter. ~3.9M ADA treasury in 2026 | Best "indexer/infra in Rust" entry point |
| **Blink Labs** | Go stack: gOuroboros, Adder, **Dingo** node (6.9M ADA 2026) | Go devs' on-ramp |
| **Sundae Labs** | SundaeSwap DEX, **treasury contracts**, Gummiworm (Hydra-based) [UNVERIFIED current status] | Reusable governance/treasury contracts |
| **HLabs (Harmonic Laboratories)** | TypeScript core: Pebble (language), Gerolamo (TS node in the browser). Pebble enacted at 4.6M ADA; Gerolamo 2026 bid failed | TS devs |
| **Tweag (Modus Create), VacuumLabs, Ensurable Systems, MLabs** | Core contractors (ledger, Plutus, maintenance, Plutarch) | — |
| **AlphaGrowth** | PRIME operator | DeFi grants and RFP |

### 4.1 Node implementations (client diversity)

| Node | Language | Org | Status (Oct 2026) | Treasury |
|---|---|---|---|---|
| cardano-node | Haskell | IO (+ Tweag / Ensurable maintenance) | Canonical. van Rossem (PV11) enacted Jul 2026; Dijkstra-era work ongoing | Maintenance 62.1M ADA |
| **Amaru** | Rust | PRAGMA | Runs as a **mainnet relay**: bootstraps from Mithril, weekly betas. **Block production target Nov 2026** [S20] | 1.5M (2025), **10.14M (2026)** |
| **Dingo** | Go | Blink Labs | Testnet-only; has block production, Plutus V1–V3, 48+ releases [S22] | **6.9M (2026)** |
| **Gerolamo** | TypeScript | HLabs | Browser node, early stage | 579K (2025); 2026 asks failed |
| **Acropolis** | Rust (modular, Caryatid) | IO | **Discontinued Apr 2026**, code still open source [S21] | 1.4M returned |
| Dolos | Rust | TxPipe | Lightweight data node (not a block producer) | 221K (2025), 541K (2026) |

---

## 5. "How do I get funded in 2026?" Decision tree

```
START: What are you building and how far along is it?

├─ Idea / hackathon stage, < $20k needed
│   → Hackathons (Midnight MLH/Buidl events, Gimbalabs Piece of Pie, Charli3-style sponsor hackathons)
│   → Genesis Pre-Accelerator (≤$20k), Midnight Build Club
│   Timeline: weeks. Amount: $1k–$20k.

├─ Early product that uses USDCx / Pyth / programmable tokens / identity, can ship in 3 months
│   → Catalyst Pilot (50k–200k ADA, 40/40/20 usage-based payouts)
│   Status: 2026 round closed 20 Aug; watch projectcatalyst.io/apply for the next one [UNVERIFIED date]
│   Timeline: ~5 weeks to decision, 3 months to mainnet.

├─ Live DeFi protocol / wallet / bridge / liquidity tooling that can grow TVL
│   → Cardano PRIME RFP (₳35M grant envelope, TVL-per-grant-dollar metric; >$1M → separate gov action)
│   Status: OPEN (since Sep 2026), deadline [UNVERIFIED]
│   Timeline: rolling during months 2–12 of the program (Aug 2026 → Aug 2027).

├─ Startup ready to raise equity/tokens
│   → Orion Fund ($80M target, acceleration → Series A), CF Cardano Accelerator Program (CHF 10k + Demo Day),
│     Apex Growth Accelerator (≤$70k / ~3.5%), Midnight Night Sky Accelerator
│   Timeline: 1–3 months.

├─ You maintain an important open-source library/tool
│   → Intersect Maintainer Retainer (recurring) → then the Intersect annual budget process
│     (min 100k ADA, 1k ADA fee, 67% off-chain DRep vote, Intersect administers via treasury contracts)
│   Timeline: retainer ~1–2 months; budget cycle ~Apr–Jul each year.

└─ Large infrastructure / protocol work (≥ ~₳1M) with ecosystem-wide impact
    → Direct treasury withdrawal (100k ADA refundable deposit; ≥67% DRep stake + CC; ~30-day vote)
    Prereqs: a named administrator, audit budget, milestones, IPFS metadata, NCL headroom (only ~43M ADA left to Jul 2027)
    Timeline: 2–4 months including socialising. Success rate 2026: ~53% (31/59).
```

| Path | Typical amount | Time to money | Open now? | Main gate |
|---|---|---|---|---|
| Hackathon | $1k–$30k | Days–weeks | Rolling | Judges |
| Catalyst Pilot | 50k–200k ADA (~$10k–$42k) | ~2 months + usage KPIs | Next round [UNVERIFIED] | Curators + 35-person panel |
| PRIME RFP | Up to $1M per grant (larger needs a separate gov action) | Rolling | **Yes** | AlphaGrowth + Operating Group (3-of-5) |
| CAP / accelerators | CHF 10k – $70k | 1–3 months | Cohort-based | Selection |
| Orion Fund VC | Seed – Series A | VC process | Yes | Investment committee |
| Maintainer Retainer | Recurring salary-like | ~1–2 months | Yes | OSC/TSC + 30-day evaluation |
| Intersect budget | ≥100k ADA | ~3 months (Apr→Jul) | Next cycle ~2027 [UNVERIFIED] | 67% off-chain DRep vote, then on-chain |
| Direct treasury withdrawal | 100k ADA → 100M+ ADA | ~1–3 months | Always (NCL-limited) | 67% DRep stake + CC |

---

## 6. Ideas for interactive website content

1. **Live Treasury Meter.** Pull Koios `/totals` each epoch to show the balance, inflow per epoch, withdrawals and NCL headroom, as an animated "fuel gauge" with a 2025→2026 history line.
2. **Governance Action Explorer + Threshold Simulator.** Browse every action with outcomes. Let users drag DRep, SPO and CC sliders (including the Always-Abstain effect) to see whether an action would ratify, using live epoch parameters.
3. **"Get Funded" decision wizard.** Five questions (stage, amount, open source?, DeFi/TVL?, timeline) map to the tree above, with deep links and live open/closed status.
4. **Treasury-contract playground.** A Preview-testnet demo of `treasury.ak`/`vendor.ak`: fund a vendor with milestones, pause and resume, sweep after expiry, all via the `@sundaeswap/treasury-funds` SDK. "Governance you can `npm install`."
5. **Node-diversity map and a "Follow the money" Sankey.** Show Haskell, Rust, Go and TS nodes with their status and treasury funding, plus a Sankey from Treasury → administrator (Intersect / self) → vendors → milestones, built from CIP-108 metadata.

---

## Sources

On-chain data: Koios API, `https://api.koios.rest/api/v1/` endpoints `totals`, `epoch_params?_epoch_no=660`, `proposal_list`, `proposal_voting_summary`, `committee_info`, `drep_epoch_summary` (queried 2026-10-07).

- [S1] Koios proposal `gov_action15atytcy8ru7mkcs8m7r8mx7k5x36t0h6grtgmak6v5wmf4nq07lsqhakceq` (NCL 500M) and `gov_action1m3xx08yv788vfxqh6nfvrjtvmqpwezsy0ggaczctkyjmttc2wmxsq4jsr7q` (NCL 350M)
- [S2] https://projectcatalyst.io/
- [S3] https://projectcatalyst.io/blog/update-on-fund15-voting
- [S4] https://projectcatalyst.io/blog/update-from-the-catalyst-team
- [S5] https://cryptoslate.com/cardanos-project-catalyst-is-changing-hands-and-the-pause-is-forcing-builders-to-face-a-brutal-funding-gap/
- [S6] https://forum.cardano.org/t/new-catalyst-pilot-fund-launching-this-summer/155306
- [S7] https://projectcatalyst.io/blog/meet-the-12-teams-selected-for-the-catalyst-pilot-2026
- [S8] https://forum.cardano.org/t/this-week-in-cardano-opportunities-to-get-involved/156275 ; https://projectcatalyst.io/apply
- [S9] https://alphagrowth.io/cardano-prime
- [S10] https://www.cardanocube.com/governance/gov_actions/gov_action122wue2k65qq8gmpz795z2axt8apka6ay6xt3pwg8jxj5yfkujmtsqvlfpu7
- [S11] https://thecryptobasic.com/2026/08/12/cardano-community-approves-120-million-ada-allocation-to-boost-defi-liquidity/
- [S12] https://dailycoin.com/cardanos-19m-defi-plan-faces-a-hard-reality-check/ ; https://en.coin-turk.com/cardano-considers-19-2-million-prime-proposal-to-boost-defi-tvl-by-200-million/
- [S13] https://www.hokanews.com/2026/09/cardano-ada-rises-as-midnight-and.html ; https://paragraph.com/@theadasignal/ada-signal-daily-delta-2026-09-13-263000
- [S14] https://en.coin-turk.com/cardano-gains-7-5yuzde-audit-reveals-defi-weaknesses-as-ada-nears-0-25/ ; https://github.com/shiodome47/cardano-note/pull/3
- [S15] https://www.fcbarcelona.com/en/club/news/4581767/fc-barcelona-launches-bara-fan-lab-a-new-digital-experience-that-connects-club-values-with-new-web3-ways-of-participation-and-innovation
- [S16] https://coingape.com/ada-price-jumps-5-as-fc-barcelona-taps-cardano-for-new-web3-fan-platform/
- [S17] https://cryptobenelux.com/altcoin-nieuws/fc-barcelona-lanceert-barca-fan-lab-op-cardano
- [S18] https://www.insideworldfootball.com/2026/09/28/barca-test-blockchain-with-barca-fan-lab-launch/
- [S19] https://cardano.org/news/2024-04-22-launch-of-pragma ; https://cardanofoundation.org/blog/cardano-foundation-blink-labs-dcspark-sundae-labs-txpipe-launch-open-source-association
- [S20] https://en.coin-turk.com/cardano-dijkstra-hard-fork-on-schedule-amaru-node-targets-november-2026/ ; https://hackmd.io/@PRAGMA-org/amaru-proposal
- [S21] https://www.iog.io/news/change-of-course-acropolis-tiered-pricing ; https://coinpedia.org/news/crypto-news-today-cardanos-iog-halts-acropolis-redirects-4-1m-ada-to-growth/
- [S22] https://forum.cardano.org/t/dingo-treasury-proposal-2026-building-a-production-ready-cardano-block-producer-in-go/153500 ; https://pkg.go.dev/github.com/blinklabs-io/dingo
- [S23] https://www.iog.io/news/cardano-nodes-evolution-towards-diversity-and-modular-design
- [S24] https://www.iog.io/news/from-convention-to-ratification-the-cardano-constitution ; https://cryptoslate.com/cardano-ratifies-blockchain-constitution-embracing-full-decentralization/
- [S25] https://www.intersectmbo.org/news/cardanos-delegate-endorsed-constitution-now-on-chain-for-dreps-and-the-icc-to-consider
- [S26] https://bitcoinist.com/cardano-39-proposals-under-new-budget-framework/
- [S27] https://forum.cardano.org/t/cf-ga-22-60-rationale-aggregate-rationale-for-39-treasury-withdrawals-to-execute-the-cardano-blockchain-ecosystem-budget-275m-ada-administered-by-intersect/147872
- [S28] https://intersectmbo.org/news/critical-integrations-technical-progress-update-and-pentad-v2 ; https://intersectmbo.org/news/cardano-critical-integrations-program-status-update
- [S29] https://cardano.org/constitution/ ; https://intersectmbo.org/news/updated-cardano-constitution-ratification-outcome-and-effective-date
- [S30] https://www.coindesk.com/tech/2026/04/23/input-output-seeks-usd46-8-million-to-bring-bitcoin-defi-scaling-upgrade-to-cardano
- [S31] https://www.cryptotimes.io/2026/05/25/cardano-pushes-ahead-with-leios-after-strong-governance-vote/
- [S32] https://www.theblock.co/amp/post/403122/cardano-foundation-cancels-2026-summit-after-treasury-funding-vote-falls-just-short
- [S33] https://intersectmbo.org/news/the-2026-constitutional-committee-elections ; https://docs.intersectmbo.org/intersect-knowledge-base/cardano-facilitation-services/cardano-governance/2026-constitutional-committee-elections/cc-election-2026-overview
- [S34] https://paragraph.com/@theadasignal/ada-signal-daily-delta-2026-09-13-263000
- [S35] https://dev.to/mrnasdog/ada-inflation-analysis-june-2026-a-capped-coin-that-still-drips-supply-1043 (search snippet; page 404 at fetch time)
- [S36] ADA ≈ $0.209 on 2026-09-13 per [S34]
- [S37] https://drepforum.substack.com/p/budget-season-opens-usdcx-node-diversity ; https://intersectmbo.org/news/intersect-update-report-102-march-13-2026
- [S38] https://docs.intersectmbo.org/intersect-knowledge-base/cardano-facilitation-services/cardano-budget-2026/2026-budget-process-timeline ; https://www.intersectmbo.org/cardano-budget-submission
- [S39] https://www.gncrypto.news/news/io-trims-2026-cardano-treasury-ask-38-9m-backs-leios-pogun/
- [S40] https://finance.yahoo.com/markets/crypto/articles/cardano-news-dreps-pogun-vote-091000617.html
- [S41] https://cexplorer.io/article/cardano-community-approves-50m-ada-for-the-draper-dragon-orion-fund
- [S42] https://github.com/SundaeSwap-finance/treasury-contracts (README)
- [S43] https://cardano.org/news/2025-07-18-smart-contract-tooling/ ; https://cardano.org/news/2025-07-08-automating-accountability/
- [S44] https://www.essentialcardano.io/article/navigating-cardano-governance-essential-tools-you-should-know
- [S45] https://docs.intersectmbo.org/cardano-governance/governance-tools/voltaire-govtool
- [S46] https://cardanofoundation.org/blog/cardano-foundation-budget-review-process-2026
- [S47] https://cexplorer.io/article/project-catalyst-fund-14-results-unveiled-fueling-cardano-s-next-wave-of-innovation
- [S48] https://projectcatalyst.io/blog/project-catalyst-fund15-goes-multi-chain ; https://midnight.network/blog/midnight-foundation-and-project-catalyst-launch-new-fund-tracks-to-accelerate-privacy-innovation
- [S49] https://cardano.org/grants-funding/ (redirect target of https://developers.cardano.org/docs/community/funding/)
- [S50] https://www.unlock-bc.com/135743/fc-barcelona-and-cardano-redefining-fan-engagement-through-blockchain/
- [S51] https://www.catalystexplorer.com/en/proposals/fc-barcelona-fan-engagement-infrastructure-cardano-f13 ; https://www.kucoin.com/news/community/ADA/6ab80d4a74fd460007c59005
- [S52] https://www.cardanocube.com/governance/gov_actions/gov_action13tfag48nf94rtjcdq7c06vhkslmxxw9h6c88sl7q5g5nnewcsvlp5u7pqqr
- [S53] https://cardanofoundation.org/blog/venture-hub-expands-new-programs
- [S54] https://cardanofoundation.org/blog/cap-fall26-applications ; https://cardano.org/news/2026-05-13-cardano-accelerator-program-accepting-applications/
- [S55] https://devex.intersectmbo.org/docs/maintainer-retainer ; https://opensourcecommittee.docs.intersectmbo.org/about/paid-open-source-model-posm/maintainer-retainer
- [S56] https://forum.cardano.org/t/maintainer-retainer-project-interest-forms-are-live/149244
- [S57] https://cardano.org/news/2026-05-08-intersect-committee-election-results-2026/
- [S58] https://opensourcecommittee.docs.intersectmbo.org/working-groups/developer-experience-working-group ; https://devex.intersectmbo.org/docs/getting-started
- [S59] https://midnight.network/blog/state-of-the-network-march-2026 ; https://midnight.network/blog/consensus-hk-2026-recap
- [S60] https://midnight.network/blog/night-sky-accelerator ; https://midnight.network/hackathon/hack-buenos-aires
- [S61] https://events.mlh.com/events/14061-midnight-hackathon-may-2026 ; https://luma.com/midnight-buildathon
- [S62] https://cardanofoundation.org/blog/cardano-summit-2025-by-the-numbers ; https://cardanofoundation.org/blog/cardano-summit-2025-journey-and-results
- [S63] https://common-salsa-913.notion.site/Charli3-Oracles-Hackathon-3426837aa36180e388ecd9987de7013f
- [S64] https://forum.cardano.org/t/builder-season-an-experiment-in-long-form-hackathons-piece-of-pie-by-gimbalabs/155950
- [S65] https://forum.cardano.org/t/digest-august-19-2026-cardano-at-token2049-rare-evo-2026-recap-urgent-governance-call-to-action-rethinking-catalyst-funding-real-world-impact-tech-wins-cip-updates-and-more/156342
- [S66] https://blockworks.com/news/cardano-commercial-arm-aims-to-jumpstart-new-startups-with-100m-in-funding
- [S67] https://www.theblock.co/post/407593/cardano-founding-entity-emurgo-steps-down-pentad-governance-role-wallet-exploit
- [S68] https://forklog.com/en/secondfi-to-shut-down-after-2-6-million-ada-theft/
- [S69] https://cardano.org/news/2026-01-06-community-digest/ ; https://sessionize.com/cardano-buidler-fest/
- [S70] https://cips.cardano.org/cip/CIP-0001
- [S71] https://hackernoon.com/midnight-opens-redemptions-for-45b-night-tokens-after-record-breaking-distribution-event
