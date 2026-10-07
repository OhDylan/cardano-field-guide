# Cardano Core Technology & Developer Toolchain: A Builder's Briefing (as of 2026-10-07)

> Research for a website aimed at software engineers coming from Ethereum, Solana or web2.
> Every non-obvious claim has a numbered source (see **Sources**). Anything I couldn't confirm is marked **[UNVERIFIED]**.
> Live chain state was checked on 2026-10-07 through the public Koios API: mainnet is in the **Conway era, protocol version 11.0, epoch 660, block ~14,035,267** [S1].

---

## 0. TL;DR for builders

- **Mental model:** Cardano is not a global key-value store with contracts that mutate it. Value and state sit in **UTxOs** (immutable outputs). A "smart contract" is a **validator**: a pure predicate that decides whether a transaction may spend a UTxO, mint a token, or withdraw rewards. Your off-chain code computes the new state and builds the transaction. The chain only *checks* it.
- **Deterministic and predictable:** you can simulate a transaction locally and get the exact fee and execution budget before you submit. There is no gas auction, and a script can't fail on-chain because some other transaction changed the state first. If an input was already spent, your transaction is simply invalid and you pay nothing.
- **Tokens are native:** minting needs a minting policy (a native script or a Plutus script), but transfers need no contract. There is no ERC-20 or approve/transferFrom. CIP-113 "programmable tokens" adds compliance hooks for stablecoins and RWAs [S40].
- **Language:** **Aiken** is the default. More than 75% of respondents in the 2025 Cardano developer survey use it [S8], and it releases often (v1.1.24, 2026-09-26) [S10].
- **Off-chain TypeScript:** **Mesh** (`@meshsdk/core` 1.9.x), **Evolution SDK** (now an IntersectMBO-incubated project, very active, pure TS, no WASM), **Blaze**, and **Lucid Evolution** (in maintenance while Evolution SDK matures) [S12–S15, S2].
- **2026 protocol changes:** the **van Rossem** intra-era hard fork (PV11, 2026-07-18) added Plutus builtins (BLS12-381 multi-scalar multiplication, arrays, modexp, faster value ops, `case` on builtin types) and made all builtins available to Plutus V1, V2 and V3 [S3, S4, S5]. Next is the **Dijkstra** era (PV12): Linear Leios, nested transactions (CIP-118), Plutus V4, and guard/observe scripts (CIP-112). The code-complete target is Q4 2026. Mainnet enactment is estimated at **Dec 5 2026–Jan 4 2027 (moderate confidence)** or **Feb 24–Mar 26 2027 (high confidence)** [S6, S7, S21]. **Peras** (faster finality) follows in Phase 2, targeted for Q2 2027 [S6].
- **Scaling:** **Hydra** reached v1.0 (Oct 2025) and v2.x (Apr–Sep 2026, current 2.4.1). It ran Midnight's Glacier Drop [S23–S26]. The **Leios** testnet ("Musashi Dojo" / MusashiNet) has been live since 2026-06-23 [S19, S21].
- **Surprises:** the **Partner Chains toolkit repo is archived** and its development has been folded into Midnight's node [S27]. **plu-ts has been rebranded "pebble"** [S28]. **Marlowe moved to the `marlowe-lang` org**, and its playground is now at `playground.marlowe-lang.org` [S29]. **Demeter.run now sells managed APIs** and its homepage no longer mentions the cloud-IDE workspaces it was known for [S30]. A **Cardano Foundation "cardano-dev-skills"** repo ships AI-agent skills for Claude Code and Codex [S31]. IOG's 3.6M ADA **Developer Experience Initiative** (cardano-init CLI, an OpenZeppelin-style contracts library, a unified portal) is due by the end of Q4 2026 [S32].

---

## 1. The mental model

### 1.1 eUTxO vs the account model, explained for an EVM developer

| Concept | Ethereum / EVM | Solana | Cardano (eUTxO) |
|---|---|---|---|
| Where state lives | Contract storage (global mutable map) | Accounts owned by programs | **UTxOs**: immutable outputs, each with an address, a value (ADA + tokens) and an optional **datum** |
| What a contract is | Code that *executes* and mutates storage | Program that mutates the accounts passed in | **Validator**: a pure function `(tx, purpose) -> ok / fail` that only checks a proposed transaction |
| Who computes the new state | The chain, during execution | The chain | **Off-chain code** computes it. The chain checks it. |
| A "call" | `contract.foo(args)` | Instruction with accounts | Spend the script UTxO with a **redeemer** (the "argument" or action) and create new outputs (the new state) |
| Contract-to-contract composition | Internal calls | CPI | **Transaction-level composition**: one tx can spend from, mint with, and reference many scripts. Each validator sees the *whole* transaction. |
| Failure mode | Revert, but you still pay gas | Fail, but you pay fees | Fails during local evaluation, before submission. A tx whose inputs are already spent is rejected for free. Collateral is lost only if you knowingly submit a script-failing tx. |
| Fees | Gas × dynamic price (auction) | Base + priority fees | `a × size + b` + `price × exUnits` + reference-script fee. Known exactly at build time. |
| Tokens | ERC-20/721 contracts | SPL Token program | **Native assets**: a ledger feature, not a contract |
| "Deploying" | Deploy tx creates a contract address | Deploy a program | **No deploy needed.** The script address is the hash of the script. You lock funds at it. Optionally publish the script once as a **reference script** so later txs don't carry the bytes. |

**The one-sentence version:** *Ethereum runs your code to find out what happens. Cardano asks your code whether what you say happens is allowed.*

**Anatomy of a spend:**
- **Datum**: data attached to a UTxO, i.e. the contract's state (for example `{ owner, deadline }`). Since Vasil it can be **inline** (stored in the output itself) rather than only a hash [S33].
- **Redeemer**: the argument supplied by whoever spends the UTxO (for example `Unlock` or `Cancel`).
- **Script context**: the full transaction (inputs, reference inputs, outputs, mint, signatories, validity interval, withdrawals, certificates, votes and proposals in V3) plus the "purpose" (spend, mint, withdraw, publish, vote, propose). In **Plutus V3**, validators take a single context argument, and datum and redeemer live inside it [S34].
- **Reference inputs (CIP-31)**: read a UTxO without consuming it, for oracles, config, or protocol parameters. **Reference scripts (CIP-33)**: put the script in a UTxO once and reference it, which shrinks txs and fees. **Inline datums (CIP-32)** [S33].
- **Validity interval**: scripts see a time *range* (not "now"). This keeps evaluation deterministic.

### 1.2 Determinism and fee predictability (current mainnet parameters)

Live values from Koios on 2026-10-07 [S1]:

| Parameter | Value | What it means |
|---|---|---|
| `txFeePerByte` / `txFeeFixed` | 44 / 155,381 lovelace | A simple ADA transfer (~300 bytes) costs ≈ 0.17 ADA |
| `executionUnitPrices` | mem 0.0577, steps 0.0000721 lovelace | Script cost is priced per unit of memory and CPU |
| `maxTxExecutionUnits` | 16.5M mem / 10B steps | Hard per-tx script budget (the Cardano equivalent of a gas limit) |
| `maxBlockExecutionUnits` | 72M mem / 20B steps | Per-block budget |
| `maxTxSize` / `maxBlockBodySize` | 16,384 B / 90,112 B | Why reference scripts matter |
| `utxoCostPerByte` | 4,310 lovelace | **Min-ADA**: every output must carry ADA proportional to its size. This surprises newcomers. |
| `minFeeRefScriptCostPerByte` | 15 | Fee for referenced script bytes. It is tiered and becomes a proper protocol parameter in Dijkstra [S6]. |
| `collateralPercentage` | 150 | Collateral posted for script txs (lost only on phase-2 failure) |
| `protocolVersion` | 11.0 | van Rossem |

Pitch line for the website: *"Simulate locally, get the exact fee, submit. The only race is 'someone spent this first', and that costs you nothing."*

### 1.3 Native assets (no contract needed)

- A token is `policyId.assetName`. The **policy ID** is the hash of the minting policy. That policy is either a **native script** (multisig and/or time-lock, no Plutus needed) or a Plutus validator with the `mint` purpose.
- Tokens move with ADA inside UTxOs. Any wallet can send them, and there is no `transfer` function to bug out. NFTs use a one-shot policy (it must consume a specific UTxO) or a time-locked policy.
- Metadata standards: **CIP-25** (tx metadata) and **CIP-68** (datum-based, updatable metadata via a reference NFT).
- **CIP-113 programmable tokens** (2026) let issuers attach rules (allowlists, freezes, royalties) to native assets, aimed at regulated stablecoins and RWAs. The Cardano Foundation pushed it forward in Feb–Mar 2026, and FluidTokens and Minswap are building a CIP-113-aware DEX [S40]. The same reports mention a **USDCx** stablecoin launch on Cardano in early 2026 [S40]; details were not deeply verified **[UNVERIFIED specifics]**.

### 1.4 The concurrency "problem" and the standard patterns

**The issue:** a UTxO can be spent only once. If your whole AMM pool is one UTxO, only one swap per block can touch it, and two users racing for it means one tx fails. SundaeSwap's 2021 post is the canonical write-up [S35].

**Standard solutions (all in production):**

| Pattern | How it works | Who uses it |
|---|---|---|
| **Batcher / order-UTxO** | Users lock "order" UTxOs at a request script. An off-chain **batcher** (SundaeSwap calls it a "scooper") aggregates many orders into one tx against the pool UTxO. Validators enforce that each order is filled at a fair price or refunded. | Minswap, SundaeSwap, most AMMs [S35, S36] |
| **Order book (orders as UTxOs)** | Each limit order is its own UTxO. Anyone can match. There's no shared state, so matching is concurrent. | Genius Yield [S37] |
| **State-thread token (STT)** | A unique NFT marks the "canonical" state UTxO. Validators require that the NFT moves to the continuing output. This prevents fake state. | Most stateful dApps |
| **Sharding / many UTxOs** | Split state (for example liquidity) across N UTxOs so N users can act in parallel. | Lending pools, faucets |
| **Withdraw-zero / stake-validator ("forwarding") pattern** | Spending validators only check that a staking script ran (via a 0-ADA withdrawal). The heavy logic runs *once per tx*, not once per input. This is a huge budget saving for batch txs. | Widely used, documented on the dev portal [S38] |
| **UTxO indexers** | Redeemer carries input→output index mappings so validators avoid O(n²) searches. | Anastasia Labs `aiken-design-patterns` [S38] |
| **Linked lists / Merkle-Patricia Forestry** | On-chain sets and maps that scale (airdrops, registries) with proofs instead of full state. | Aiken MPF library **[UNVERIFIED current version]** |
| **Hydra L2** | Move high-frequency interaction off L1 entirely (§6). | DeltaDeFi, Glacier Drop [S24, S26] |
| **Nested transactions (coming in Dijkstra)** | Partial/unbalanced sub-transactions that a third party completes and batches. This enables intent-style swaps and **Babel fees** (paying fees in non-ADA tokens) without two-way coordination. | CIP-118 [S39] |

Honest framing: concurrency is a design constraint, not a blocker. It pushes you toward intent/order-based architectures, which is roughly where Ethereum went anyway with CoW, UniswapX and 1inch Fusion.

### 1.5 Ouroboros Praos (brief)

- Proof-of-stake. ~3,000 community stake pools; ADA holders delegate without lock-up or slashing. Slot leaders are chosen by a private **VRF** lottery.
- 1-second slots and an active-slot coefficient of 0.05 give a block roughly every **20 s**. An epoch is **5 days** (432,000 slots). The security parameter is k = 2160. Finality is probabilistic, which is why **Peras** exists (§6) **[standard params; not re-verified this session]**.
- Node diversity is arriving. **Amaru** (Rust, built by the PRAGMA consortium: Cardano Foundation, Blink Labs, dcSpark, Sundae Labs, TxPipe) already runs as a mainnet relay and targets block production in **November 2026** [S41]. Other node efforts discussed in Intersect's hard-fork working group: Dingo (Go), Dugite (Rust), Gerolamo (TS), Dolos (Rust data node), Scalus (Scala) and more [S7].

### 1.6 Eras and hard forks: what each one gave developers

| Hard fork | Date | PV | Dev-relevant changes |
|---|---|---|---|
| Mary | Mar 2021 | 4 | Native tokens [S2] |
| Alonzo | Sep 2021 | 5 | Plutus smart contracts (V1) [S2] |
| Vasil | Sep 2022 | 7 | Plutus V2, reference inputs, inline datums, reference scripts (CIP-31/32/33) [S2, S33] |
| Valentine | Feb 2023 | 8 | SECP256k1 builtins (ECDSA/Schnorr) [S2] |
| **Chang** | **2024-09-01** | 9 | Conway era: **Plutus V3** (single-argument scripts, governance script purposes, BLS12-381 (17 primitives), Blake2b-224, Keccak-256, sums-of-products encoding), plus the first CIP-1694 governance features [S2, S34] |
| **Plomin** | **2025-01-29** | 10 | Full on-chain governance (DReps, all governance actions). It was intra-era, and new builtins (bitwise ops CIP-122/123, RIPEMD-160 CIP-127) became available to V2/V3 [S2, S42]. The exact builtin set per PV is **[UNVERIFIED]**. |
| **van Rossem** | **2026-07-18** (epoch 644) | 11 | Intra-era. **All builtins unified across Plutus V1/V2/V3.** **`case` on Bool/Integer/Data in UPLC** (big pattern-matching speedup). New builtins: **BLS12-381 multi-scalar multiplication (CIP-133), native arrays (CIP-138), modular exponentiation (CIP-109), `dropList` (CIP-132), MaryEraValue/value ops (CIP-153)**. Revised reference-input rules for V1/V2. VRF-key uniqueness. A cost-model update on 2026-06-18 priced the new builtins and **reduced costs for existing operations**. It was the first hard fork proposed, debated and ratified fully via CIP-1694: 77.6% DRep, 52.7% SPO, plus Constitutional Committee [S3, S4, S5]. |
| **Dijkstra Phase 1** | Target Q4 2026 code-complete. Mainnet est. Dec 2026–Jan 2027 (moderate) or Feb–Mar 2027 (high) | 12 | New era. **Ouroboros Linear Leios (CIP-164)**, **Nested transactions (CIP-118)**, **Observe/guard scripts (CIP-112)**, **Plutus V4 script context** (expected ready Sep 2026), account-address groundwork (CIP-159), removal of `isValid` (CIP-167), non-segregated block body (CIP-176), fee change for reference inputs, reference-script pricing as protocol parameters, no DRep needed to withdraw rewards (CIP-181), CIP-50 pledge leverage (dormant by default) [S6, S7, S21] |
| **Dijkstra Phase 2** | Target Q2 2027 | 12.x intra-era | **Ouroboros Peras** (CIP-140) and possibly Fair Min Fees (CIP-23) [S6] |

Deferred out of Dijkstra: CIP-156 (multiIndexArray builtin) and CIP-160 (receiving script purpose) [S7].

Testnet path for Dijkstra: **DijkstraNet** (public launch with node 11.2) for Plutus V4 and nested txs, and **MusashiNet** for Leios. After that it goes Preview → Preprod → Mainnet, each with a governance action [S21].

---

## 2. Smart-contract languages (on-chain)

All of these compile to **UPLC** (Untyped Plutus Core). The chain doesn't care which language you used [S9].

| Language | Host syntax | Latest release (checked 2026-10-07) | Maturity / adoption | Best for |
|---|---|---|---|---|
| **Aiken** | Rust/Gleam-like, purpose-built | **v1.1.24 (2026-09-26)**; stdlib v3.x [S10] | **>75% of surveyed devs use it** [S8]. Accepted into GitHub Linguist [S8]. Built-in test runner, property tests, benchmarks, blueprint (CIP-57) output, LSP | **Default recommendation** |
| **Plinth** (formerly Plutus Tx) | Haskell (GHC plugin) | plutus 1.71.0.0 (2026-09-30) [S43] | Original IOG/Intersect framework, renamed early 2025 to separate it from Plutus Core [S44] | Haskell teams, high-assurance infra |
| **Plutarch** | Haskell eDSL | v1.10.1 (Feb 2025), repo still active Sep 2026 [S43] | Very efficient bytecode, steep curve [S9] | Hyper-optimized protocols |
| **Scalus** | Scala 3 (JVM/JS/native) | v1.2.0 (2026-09-17) [S43] | Lantr Engineering. Includes an in-memory **ledger emulator** (Mesh uses it for offline testing [S13]) and early Dijkstra features. 2026–27 treasury proposal for ₳8.5M [S45] | JVM/enterprise teams |
| **OpShin** | Python subset (valid Python) | 0.28.0 (2026-09-01) [S43] | Production-usable. Lower performance [S9] | Python devs, prototyping |
| **Helios** | Own curly-brace DSL, compiler in JS/TS | `@helios-lang/compiler` 0.17.33 (2026-09-08) [S43] | Pre-1.0. Compiles in the browser with no toolchain [S9, S46] | Pure front-end/JS teams |
| **pebble** (formerly **plu-ts**) | TS-like language | `@harmoniclabs/pebble` 0.4.4 (2026-08-05). Last `plu-ts` release 0.9.0 (Jan 2025) [S28] | Harmonic Labs rebrand to a standalone language "with an imperative bias" | TS devs. Watch maturity. |
| **Marlowe** | Non-Turing-complete financial DSL plus Blockly | Repo now at `marlowe-lang/marlowe-cardano` (active Oct 2026) [S29] | Niche. Playground, runtime and TS SDK still exist | Escrows, simple financial contracts, non-programmers |
| **Tx3** (not on-chain) | Declarative **transaction-template** DSL by TxPipe | `tx3-lang` 0.2x | Describes a dApp's *transactions* (not validators). Generates typed clients (TS/Rust/Go/Python). Registry at **tx3.land** [S47, S48] | Publishing dApp interfaces |

### Recommendation for newcomers in 2026: Aiken

1. It's the de facto standard: >75% usage [S8]. Most audits, tutorials, AI-agent skills (the Cardano Foundation's `cardano-dev-skills` defaults to Aiken [S31]) and design-pattern libraries target it.
2. Everything comes in one binary: `aiken new`, `aiken check`/`aiken test` (unit and property tests with coverage), `aiken bench`, `aiken build` (emits a CIP-57 **blueprint** JSON that off-chain SDKs consume), `aiken docs`, LSP, and the browser **playground** at play.aiken-lang.org [S10, S49].
3. It tracks protocol changes quickly. v1.1.23 and v1.1.24 added PV11 cost models and first-class `Value` support with value literals within weeks of van Rossem [S10].
4. It's familiar to Rust and TypeScript developers: expression-oriented, pattern matching, no monads.

### What an Aiken validator looks like (from the official Hello World, Plutus V3 style) [S49]

```aiken
use aiken/collection/list
use aiken/crypto.{VerificationKeyHash}
use cardano/transaction.{OutputReference, Transaction}

pub type Datum {
  owner: VerificationKeyHash,
}

pub type Redeemer {
  msg: ByteArray,
}

validator hello_world {
  spend(datum: Option<Datum>, redeemer: Redeemer, _own_ref: OutputReference, self: Transaction) {
    expect Some(Datum { owner }) = datum
    let must_say_hello = redeemer.msg == "Hello, World!"
    let must_be_signed = list.has(self.extra_signatories, owner)
    must_say_hello? && must_be_signed?
  }

  else(_) {
    fail
  }
}

test hello_world_example() {
  let datum = Datum { owner: #"00000000000000000000000000000000000000000000000000000000" }
  let redeemer = Redeemer { msg: "Hello, World!" }
  let utxo = OutputReference { transaction_id: "", output_index: 0 }
  hello_world.spend(Some(datum), redeemer, utxo,
    Transaction { ..transaction.placeholder, extra_signatories: [datum.owner] })
}
```

Note for EVM developers: there's no storage write and no `msg.sender`. The validator inspects the *proposed* transaction (`extra_signatories`) and returns true or false. The `?` operator traces which condition failed.

---

## 3. Off-chain and client SDKs

npm data was pulled from the registry on 2026-10-07 [S12]. Weekly downloads are a rough signal only, because they include transitive dependencies.

| SDK | Language | Latest (date) | npm weekly dl | Status / notes |
|---|---|---|---|---|
| **Mesh** (`@meshsdk/core`) | TS/JS | 1.9.1 (2026-06-19) | ~10.0k | Very active. Most beginner-friendly. React components, CIP-30 wallets, `MeshTxBuilder`, providers (Blockfrost, Koios, Maestro, Ogmios, UTxO RPC, Yaci), Aiken integration, Hydra provider, contract library (escrow, vesting, marketplace, swap), Scalus emulator for offline tests, `npx meshjs` scaffolding, AI skills, MCP server, llms.txt [S13, S50] |
| **Evolution SDK** (`@evolution-sdk/evolution`) | TS (Effect-based) | 0.5.17 (2026-10-05) | ~4.3k | **First project in Intersect's incubation program** (IntersectMBO/evolution-sdk), by No Witness Labs. Pure TS with **no WASM or CML**. Runs in Node, Bun, Deno, browser and edge. Providers: Blockfrost, Maestro, Koios, Kupo/Ogmios. Pre-1.0 but releasing several times a week [S14, S15] |
| **Lucid Evolution** (`@lucid-evolution/lucid`) | TS | 0.6.7 (2026-10-05) | ~9.1k | Fork of the original Lucid (which broke at Chang). README: *"maintained fork while we prepare for Evolution SDK 2.0 … bug fixes and stability"*. Used by Liqwid, Indigo, WingRiders [S16] |
| **Blaze** (`@blaze-cardano/sdk`) | TS | 0.3.1 (2026-07-21); `catalyst` tag 1.0.0 | ~12.6k | Butane team. Includes an emulator. Yaci Store integration guide [S17] |
| **cardano-js-sdk** (`@cardano-sdk/core`) | TS | 0.47.1 (2026-09-25) | n/a | IOG. Powers Lace. Lower-level. |
| **cardano-serialization-lib** (CSL) | Rust→WASM/JS | 17.0.0 (2026-08-07) | n/a | EMURGO. The foundational serialization lib. |
| **cardano-multiplatform-lib** (CML) | Rust→WASM | 6.2.0 (2025-04-07) | n/a | dcSpark. Release cadence has slowed. |
| **PyCardano** | Python | v0.19.2 (2026-03-01), repo active Oct 2026 [S43] | n/a | The Python standard. Pairs with OpShin. |
| **Cardano Client Lib (CCL)** | Java/JVM | v0.7.2 (2026-05-21), repo active [S43] | n/a | BloxBean. Same team as Yaci DevKit and Yaci Store. |
| **Atlas** | Haskell | v0.14.1 (2025-05-21), last push Feb 2026 [S43] | n/a | Genius Yield. **Slowing**, so check before adopting. |
| **Scalus** | Scala/JVM/JS | v1.2.0 | n/a | On-chain and off-chain in one language, plus emulator [S45] |
| **UTxO RPC SDKs** | Py/Go/Rust/Node/.NET/Deno | `@utxorpc/sdk` 0.9.0 (2026-08-27) | n/a | gRPC spec (TxPipe). Served by Dolos and Demeter [S51] |
| **Tx3 clients** | TS/Rust/Go/Python (Java/Swift soon) | n/a | n/a | Generated from `.tx3` templates [S47] |
| **Marlowe TS SDK / Runtime** | TS | n/a | n/a | Niche |
| *Original Lucid, Nami wallet* | n/a | n/a | n/a | **Dead.** Lucid broke at Chang [S16]. Nami was folded into Lace (namiwallet.io redirects to lace.io, checked 2026-10-07). Older tutorials still reference both, which is a classic pain point. |

**Recommendation:** for TypeScript newcomers, **Mesh** (largest docs, playground-style guides, React components, templates). For teams wanting strict typing and a long-term "blessed" stack, **Evolution SDK**, which Cardano Foundation's scaffold skill now uses [S15]. Python developers should use **PyCardano**. JVM developers should use **CCL** and **Scalus**.

Wallet connection is standardized via **CIP-30** (dApp↔wallet API) and **CIP-95** (governance extension). Main wallets: Lace, Eternl, VESPR, Typhon, Yoroi **[wallet list not re-verified]**.

---

## 4. Infrastructure and APIs

| Tool | Type | Status (2026) | Notes |
|---|---|---|---|
| **Blockfrost** | Hosted REST API (+ IPFS) | Active. Self-hostable "RYO" backend v6.8.0 (2026-09-02) [S43] | Free tier ≈ **50k requests/day**, 10 rps (burst 500). Mainnet, preprod and preview each need a separate key [S52, S53] |
| **Koios** | Decentralized, community-run REST API | Active | **Usable without an API key** for light use. A public tier with optional free key gives higher limits **[tier numbers UNVERIFIED]**. Mainnet, preprod, preview. Great for live website widgets (I used it for §1.2) [S1, S54] |
| **Maestro** | Hosted API (Cardano + Bitcoin) | Active, but strategic focus has shifted toward Bitcoin/BitcoinOS. Cardano still supported [S55] | Evaluate longevity |
| **Ogmios** | WebSocket JSON-RPC bridge to node | **v7.0.0 (2026-06-20)**: preliminary Dijkstra and Plutus V4 support, breaking schema changes [S56] | Used by most TS SDKs for evaluation and submission |
| **Kupo** | Lightweight UTxO/datum indexer | v2.12 (2026-07-18) [S43] | Pattern-matching queries |
| **Dolos** (TxPipe) | Lightweight "data node" | v1.6.1 (2026-09-17) | Serves **Blockfrost-compatible REST (MiniBF)**, Mini-Kupo, UTxO RPC (gRPC), Tx3 TRP, and N2C socket. Ledger-only, sliding-window or archive modes [S57] |
| **Oura** (TxPipe) | Event streaming pipeline (node → Kafka/webhooks/etc.) | v2.2.0 (2026-06-15) [S43] | |
| **UTxO RPC** | gRPC spec | Active [S51] | Streaming, binary, cross-language |
| **Yaci Store** (BloxBean) | Modular Java indexer with Blockfrost-compatible API | v2.0.3 (2026-09-23) [S43] | |
| **Cardano DB Sync** | Full chain to PostgreSQL | 13.7.2.1 (2026-06-17) [S43] | Heavy. Use for analytics and explorers. |
| **Demeter.run** (TxPipe) | Managed infra | Now offers **managed Blockfrost-compatible (Dolos), Ogmios, Kupo, UTxO RPC** on mainnet, preprod and preview. Free access program for OSS, pre-revenue and hackathon teams [S30] | The 2023–24 browser "workspaces" are no longer on the homepage **[UNVERIFIED whether discontinued]** |
| **Mithril** | Stake-certified snapshots | Repo moved to IntersectMBO. Used for fast node bootstrap (minutes, not days) and by Amaru [S41, S58] | Light-client foundation |
| **cardano-node / cardano-cli** | Reference node | 11.1.3 (2026-09-29). 11.2 (full Dijkstra features minus Leios) and 11.3 (hard-fork release candidate) are coming [S21, S43] | |

### Local devnets and emulators

| Tool | What | How |
|---|---|---|
| **Yaci DevKit** | One-command local Cardano devnet. Sub-second blocks (100–200 ms possible). Blockfrost-compatible API via Yaci Store. Optional Ogmios and Kupo. `topup` faucet. Yaci Viewer UI on :5173. Ships via Docker, ZIP, or **npm** (good for CI) | `create-node -o --start`, then `topup <addr> <ada>` [S59]. v0.12.0-beta5 (Jun 2026) [S43] |
| **Scalus emulator** | In-memory ledger (JVM/JS/TS). Mesh uses it | Library [S45] |
| **Blaze emulator** | In-memory emulator in TS | Library [S17] |
| **Evolution SDK DevNet** | DevNet integration | [S14] |
| **Hydra offline mode** | Single-party head as a fast sandbox | [S25] |
| **Aiken `check` / `bench`** | Unit and property tests against a placeholder tx; no chain needed | [S10] |

---

## 5. Networks, faucets, explorers

| Network | Purpose | Epoch | Status |
|---|---|---|---|
| **Mainnet** | Production | 5 days | Conway, PV 11.0 [S1] |
| **Preprod** | Mirrors mainnet. Final validation before launch. Hard forks within about one epoch of mainnet. | 5 days | Live [S60] |
| **Preview** | Hard forks land here first (≥ 4 weeks before mainnet). "Move fast, break things." | **1 day** | Live. **Recommended for first deploys** [S60] |
| **SanchoNet** | Governance testbed | n/a | **Still live** at sancho.network per cardano.org ("remains the place where governance features arrive first") [S61]. Some secondary sources wrongly call it retired. |
| **MusashiNet / "Musashi Dojo"** | Ouroboros Leios testnet (SPO-focused, incentivized, 5 phases: Earth→Water→Fire→Wind→Void) | n/a | Live since 2026-06-23 [S19, S21] |
| **DijkstraNet** | Plutus V4, nested txs, CIP-50 | n/a | Public launch expected with node 11.2 (≈ Oct 2026) [S21] |

- **Faucet:** https://docs.cardano.org/cardano-testnets/tools/faucet gives **10,000 tADA per request, once per 24 h**, for Preview and Preprod. An API key unlocks larger allocations [S62, S63]. Yaci DevKit's `topup` is unlimited locally.
- **Explorers:** **Cardanoscan** (cardanoscan.io; preview.cardanoscan.io and preprod.cardanoscan.io), **Cexplorer** (cexplorer.io, preview.cexplorer.io), **AdaStat** (adastat.net, preview.adastat.net), and the hub explorer.cardano.org (all checked live 2026-10-07; Cardanoscan returns 403 to bots but works in a browser). **tx3.land** shows live per-protocol activity [S48].

---

## 6. Scaling and roadmap technology

| Tech | What it is | Status (Oct 2026) | Dev impact |
|---|---|---|---|
| **Hydra Head** | Isomorphic state channels: the same ledger rules and Plutus scripts run in an L2 "head" among known participants, then fan out to L1 | **v1.0.0 (2025-10-08)** declared production-ready [S23, S25]. **v2.0.0 (2026-04-05)** removed the commit phase (heads open directly, funds added via incremental deposits), which fixed "non-abortable head" issues and lowered lifecycle cost. It also removed the 100 ADA mainnet commit cap [S25]. Latest **2.4.1 (2026-09-02)** [S43]. | Real deployments: **Midnight Glacier Drop** (heads run by Alchemy, Anastasia Labs, BitGo, Blockdaemon, IOE, SundaeLabs) [S26]. **DeltaDeFi, Masumi, Blockfrost, VTech Labs** named as live users [S24]. Mesh has a `@meshsdk/hydra` provider [S12]. Best for known-participant, high-frequency apps (payments, games, order matching). |
| **Ouroboros Leios (Linear Leios, CIP-164)** | Endorser blocks certified by stake-pool committees alongside Praos blocks | Testnet live (Jun 2026). The first testnet hit ~6× current throughput. The FAQ claims up to **1,500+ TPS (30–65×)**. Ships in Dijkstra Phase 1, with throughput raised gradually via parameters afterward. IOG treasury ask ₳27.7M [S18, S19, S20, S7] | Mostly transparent to dApps. Expect cheaper and faster inclusion under load. Indexers and explorers need updates for new block types [S20]. |
| **Ouroboros Peras (CIP-140)** | Stake-pool voting overlay for fast settlement | Dijkstra Phase 2, target **Q2 2027** [S6] | Settlement in minutes instead of a long probabilistic wait. Matters for bridges, exchanges and L2s [S22]. |
| **Nested transactions (CIP-118)** | Sub-transactions with their own witnesses, completed by a batch top-level tx | Dijkstra Phase 1. Implementation "well advanced" [S7, S39] | Native intents, atomic multi-party swaps, **Babel fees** (pay fees in any token via a third party) |
| **Plutus V4 / guard scripts (CIP-112)** | New script context and an "observe" script purpose | Dijkstra Phase 1 [S7] | Cleaner transaction-level checks. Replaces the withdraw-zero hack over time. |
| **Mithril** | Stake-based threshold signatures over chain snapshots | In production for node bootstrap. Used by Amaru [S41, S58] | Light clients and bridges |
| **Partner Chains SDK** | Substrate-based sidechains secured by Cardano SPOs | **Toolkit repo archived (Apr 2026). "Partner chains is now directly part of Midnight."** [S27] | Don't recommend it as a standalone SDK anymore |
| **ZK-friendly crypto** | BLS12-381 (V3, Chang) + **multi-scalar multiplication** + **modexp** (van Rossem) + Keccak-256 / Blake2b-224 / RIPEMD-160 / SECP256k1 | Live on mainnet [S3, S34] | Groth16/PLONK verifiers on-chain, Ethereum signature checks, Bitcoin-compatible hashes. Aiken 1.1.24 fixed MSM argument conversion [S10]. |
| **Plutus performance** | SOP encoding (V3). **`case` on builtins** and arrays (PV11). Cost-model cut (Jun 2026). UPLC optimizer/certifier (avg −10% exec cost, −2% size on mainnet scripts) | Live / ongoing [S5, S64] | Cheaper scripts. More logic fits per tx. |
| **IBC** | Cardano↔Cosmos IBC incubator (Cardano Foundation) | Active (Aug–Oct 2026 commits) [S43] | **[UNVERIFIED production status]** |

---

## 7. "Try it now": playgrounds, tutorials, starter kits

| Resource | URL | What you can do |
|---|---|---|
| **Aiken Playground** | https://play.aiken-lang.org | Write, compile and test Aiken in the browser. No install. |
| **Aiken docs / Hello World** | https://aiken-lang.org | Full tutorial with Mesh, PyCardano or cardano-cli off-chain [S49] |
| **Mesh** | https://meshjs.dev | Interactive API docs, React components, `npx meshjs <app>` templates, Cardano course at /resources/cardano-course, **Mimir** RAG chatbot at mimir.meshjs.dev, MCP server and agent skills at /ai [S50] |
| **Evolution SDK docs** | https://intersectmbo.github.io/evolution-sdk/ | TS-first docs and DevNet [S14] |
| **Marlowe Playground** | https://playground.marlowe-lang.org | Blockly, TS or Marlowe contract simulation (the old play.marlowe.iohk.io no longer resolves) [S29] |
| **tx3.land** | https://tx3.land | Browse real protocol interfaces (SundaeSwap, Indigo, Hydra) and get generated SDKs [S48] |
| **Yaci DevKit** | https://devkit.yaci.xyz | Local chain in one command [S59] |
| **Demeter** | https://demeter.run | Hosted Ogmios, Kupo, UTxO RPC and Blockfrost-compatible endpoints [S30] |
| **Developer Portal** | https://developers.cardano.org | Official portal: design patterns, tool pages, office-hours videos [S38] |
| **AI agent skills** | github.com/cardano-foundation/cardano-dev-skills | 18 skills (write-validator, build-transaction, explain-eutxo, setup-devnet…) for Claude Code and Codex, with docs auto-refreshed weekly [S31] |
| **Learning** | Cardano Academy (Cardano Foundation), Gimbalabs PBL, Plutus Pioneer Program | [S65] |

### Time to first deployed contract on Preview: estimate and best path

The Intersect DevEx pathway estimates **1 day** for a first transaction and **about 1 week** for a "first Aiken contract" (learning-inclusive) [S65]. IOG's DX initiative targets **"zero to working dApp on testnet in under two weeks"** [S32].

**My estimate for an experienced EVM or web2 developer following the best path: about 45–90 minutes to lock and unlock funds at an Aiken validator on Preview. Most of that time is setup and the faucet.** **[estimate, not benchmarked]**

1. **(5 min)** Install Aiken (`aikup`, npm, or brew). `aiken new me/hello && aiken check && aiken build` produces `plutus.json`.
2. **(5 min)** Get a **Blockfrost Preview key** (or use Koios keyless) and generate a wallet with Mesh or Evolution (or a Lace testnet wallet).
3. **(5–10 min)** Request **10,000 tADA** from the faucet.
4. **(20–40 min)** Write a ~50-line TS script with Mesh `MeshTxBuilder` or Evolution SDK. Load the compiled code from `plutus.json`, derive the script address, **lock** 10 tADA with an inline datum, then **unlock** with the redeemer, collateral and required signer.
5. **(1 min)** View both txs on preview.cexplorer.io or preview.cardanoscan.io.

The "deploy" step is just sending funds to the script's hash address. Optionally, publish a reference script UTxO first. **For the website: this is a great "aha" moment to make playable.**

Upcoming: **cardano-init** (create-cardano-app style CLI with selectable stacks and AI integrations), an **audit-ready contracts library** (5 templates, OpenZeppelin-like), and a **restructured Developer HUB**, all due **end of Q4 2026** [S32, S66]. **[Not yet released as of research date: UNVERIFIED]**

---

## 8. Honest DX pain points (and whether they're still true in 2026)

| Pain point | Still true? | Evidence / nuance |
|---|---|---|
| **Fragmented tooling, no "blessed path"** | **Yes, it's the #1 complaint.** | 109-builder survey: "fragmented tooling, scattered documentation, lack of coordination, steep learning curve". "Multiple competing entry points and tech stacks confuse newcomers" [S32, S67]. Example: three or four active TS SDKs (Mesh, Evolution, Lucid Evolution, Blaze). Mitigation: cardano-init and the Developer HUB in Q4 2026. |
| **Stale tutorials** | **Yes.** | Guides still point to Nami, Flint, original Lucid, play.marlowe.iohk.io and V2-style validators. Even the 2026 Intersect pathway page lists Nami and Flint and the old Marlowe URL [S65]. |
| **eUTxO learning curve** | **Partly.** | Still the biggest conceptual hurdle [S67], but Aiken plus design-pattern libraries and AI skills (explain-eutxo) soften it. |
| **"You need Haskell"** | **No.** | Aiken dominates. Python, Scala, TS and JS options exist [S8, S9]. Haskell is only needed for Plinth or Plutarch. |
| **Concurrency** | **Solved by patterns.** | Batchers, order books and sharding are mature. Nested txs (Dijkstra) and Hydra add native options. Still a design tax. |
| **Script size and budget limits** | **Improving.** | 16 KB tx limit and per-tx budgets. Reference scripts, withdraw-zero, PV11 `case`/arrays/value builtins and cost-model cuts help [S1, S5]. |
| **Min-ADA, collateral, UTxO selection weirdness** | **Yes (inherent).** | SDKs hide most of it, but it still surprises EVM developers. |
| **Small developer pool** | **Yes.** | IOG: about "17× fewer developers than Ethereum, gap widening" [S32]. |
| **Breaking changes at hard forks** | **Yes, at era boundaries.** | Chang broke original Lucid [S16]. Ogmios v7 has breaking schema changes ahead of Dijkstra [S56]. Intra-era forks (Plomin, van Rossem) were low-impact by design [S3]. |
| **Indexer and API choice confusion** | **Partly.** | The Blockfrost-compatible API is becoming a de facto standard (Blockfrost, Dolos MiniBF, Yaci Store, Demeter), which reduces lock-in [S30, S57, S59]. |
| **Slow finality and low TPS** | **Being addressed.** | Leios (Dijkstra, about end of 2026) and Peras (2027). Hydra is available now. |
| **Lack of an OpenZeppelin equivalent** | **Yes, for now.** | Explicitly called out. A contracts library is in progress [S66]. Mesh ships some contracts [S50]. |

---

## 9. Ideas for interactive website content (core tech)

1. **"Account vs UTxO" side-by-side simulator.** The same "counter / token transfer / AMM swap" shown as EVM storage mutation vs UTxOs consumed and created. Let users trigger two simultaneous swaps and watch one get rejected for free, then toggle "batcher mode."
2. **Live fee calculator.** Pull live protocol parameters from Koios (keyless) and let users drag tx size, exUnits and reference-script size to see the exact fee and min-ADA. Contrast it with a gas-auction chart.
3. **Embedded Aiken sandbox.** Embed or link play.aiken-lang.org with a guided "make this validator pass" puzzle series (signature check → deadline → datum update with an STT).
4. **Lock & unlock on Preview in the browser.** CIP-30 wallet connect (Lace/Eternl) plus Mesh or Evolution. A pre-compiled Hello-World validator, faucet link, one click to lock, one to unlock, and explorer links. Show a stopwatch for "time to first contract."
5. **Roadmap timeline.** Chang → Plomin → van Rossem → Dijkstra (Leios, nested txs, Plutus V4) → Peras, with a "what this unlocks for you" card per item and a live current-epoch/PV badge from Koios.

---

## Sources

- [S1] Koios mainnet API (protocol params and tip, queried 2026-10-07): https://api.koios.rest/api/v1/cli_protocol_params, https://api.koios.rest/api/v1/tip
- [S2] Cardano.org hard forks list: https://cardano.org/hardforks
- [S3] Cardano.org glossary, van Rossem: https://cardano.org/glossary/van-rossem/
- [S4] CoinDesk, "Inside Cardano's van Rossem hard fork" (2026-07-20): https://www.coindesk.com/markets/2026/07/20/inside-cardano-s-van-rossum-hard-fork-and-what-it-means-for-users ; CryptoBriefing ratification: https://cryptobriefing.com/cardano-van-rossem-hard-fork-ratified/ ; CryptoTimes: https://www.cryptotimes.io/2026/07/20/cardano-activates-van-rossem-hard-fork-first-approved-fully-onchain/
- [S5] Intersect Cardano Upgrades knowledge base (van Rossem overview; Dijkstra overview), full corpus: https://cardanoupgrades.docs.intersectmbo.org/llms-full.txt ; https://cardanoupgrades.docs.intersectmbo.org/van-rossem-upgrade
- [S6] Intersect Product Committee, Dijkstra phased rollout plan: https://product.cardano.intersectmbo.org/hardfork-planning/dijkstra/
- [S7] Intersect Dijkstra upgrade overview (in-scope CIPs, deferred CIPs, node diversity notes): https://cardanoupgrades.docs.intersectmbo.org/dijkstra-era-upgrade/dijkstra-upgrade-overview
- [S8] Cardano Foundation, 2025 Developer Ecosystem Survey (Aiken >75%): https://cardanofoundation.org/blog/2025-developer-ecosystem-survey-results ; Aiken 2025 report context: https://cardanofoundation.org/blog/aiken-the-future-of-smart-contracts
- [S9] Intersect DevEx WG Q1 2026, Smart Contracts & Languages session notes: https://devex.intersectmbo.org/docs/working-group/q1-2026/sessions/13-smart-contracts-languages/session-notes
- [S10] Aiken CHANGELOG (v1.1.17–v1.1.24): https://github.com/aiken-lang/aiken/blob/main/CHANGELOG.md
- [S12] npm registry (versions and dates queried 2026-10-07): https://registry.npmjs.org/@meshsdk/core , /@evolution-sdk/evolution , /@lucid-evolution/lucid , /@blaze-cardano/sdk , /@cardano-sdk/core , /@emurgo/cardano-serialization-lib-nodejs , /@dcspark/cardano-multiplatform-lib-nodejs , /@helios-lang/compiler , /@harmoniclabs/plu-ts , /@harmoniclabs/pebble , /@utxorpc/sdk , /@meshsdk/hydra ; downloads: https://api.npmjs.org/downloads/point/last-week/
- [S13] Mesh GitHub: https://github.com/MeshJS/mesh
- [S14] Evolution SDK GitHub and docs: https://github.com/IntersectMBO/evolution-sdk , https://intersectmbo.github.io/evolution-sdk/ ; Essential Cardano article: https://www.essentialcardano.io/article/evolution-sdk-a-new-era-for-cardano-development ; Intersect incubation: https://committees.docs.intersectmbo.org/intersect-open-source-committee/policies/project-incubation-process/incubated-projects/evolution-sdk-no-witness-labs
- [S15] cardano-dev-skills PR "scaffold-project: the Evolution stack uses @evolution-sdk/evolution": https://github.com/cardano-foundation/cardano-dev-skills/pull/113
- [S16] Lucid Evolution repo (maintenance notice): https://github.com/no-witness-labs/lucid-evolution
- [S17] Blaze: https://blaze.butane.dev ; Blaze + Yaci Store guide: https://devex.intersectmbo.org/docs/guides/blaze-yaci-store-integration
- [S18] Cardano.org, "Cardano is ready to grow. Leios is how it gets there" (2026-05-14): https://cardano.org/news/2026-05-14-cardano-is-ready-to-grow/
- [S19] CryptoBriefing, Leios testnet "Musashi Dojo": https://cryptobriefing.com/cardano-to-launch-leios-testnet-under-musashi-dojo-name/ ; CoinAlertNews: https://coinalertnews.com/news/2026/06/23/cardano-leios-testnet-musashi-dojo ; Cryptonomist: https://en.cryptonomist.ch/2026/06/16/cardano-ouroboros-leios-milestone/
- [S20] Leios FAQ: https://leios.cardano-scaling.org/docs/faq/ ; IOG Leios page: https://cardano-engineering.iog.io/solutions/leios
- [S21] Intersect weekly update #127 (2026-09-05): https://intersectmbo.org/news/intersect-weekly-update-127-sep-5-2026
- [S22] Peras: CIP-140 https://cips.cardano.org/cip/CIP-140 ; Cexplorer explainer: https://cexplorer.io/article/understanding-ouroboros-peras
- [S23] IOG, "Scaling Cardano applications with Hydra" (Hydra v1, 2025-10-27): https://www.iog.io/news/scaling-cardano-applications-with-hydra
- [S24] IOG Hydra solution page (roadmap, live users): https://cardano-engineering.iog.io/solutions/hydra ; https://labs.iog.io/solutions/hydra
- [S25] Hydra CHANGELOG (v1.0.0 → v2.4.1): https://github.com/cardano-scaling/hydra/blob/master/CHANGELOG.md ; docs: https://hydra.family/head-protocol/
- [S26] Midnight, "Meet the ecosystem partners operating Hydra Heads": https://midnight.network/blog/partners-operating-hydra-heads
- [S27] Partner Chains repo (archived notice): https://github.com/input-output-hk/partner-chains
- [S28] HarmonicLabs pebble (formerly plu-ts): https://github.com/HarmonicLabs/plu-ts (redirects to HarmonicLabs/pebble) ; docs https://pluts.harmoniclabs.tech
- [S29] Marlowe: https://marlowe-lang.org , playground https://playground.marlowe-lang.org , repo https://github.com/marlowe-lang/marlowe-cardano
- [S30] Demeter.run homepage: https://demeter.run
- [S31] Cardano Foundation cardano-dev-skills: https://github.com/cardano-foundation/cardano-dev-skills
- [S32] IOG, "Developer experience initiative" (2026-05-18): https://www.iog.io/news/developer-experience-initiative ; https://momentum.cardano.iog.io/proposals/developer-experience
- [S33] CIPs 31/32/33 (reference inputs, inline datums, reference scripts): https://cips.cardano.org/cip/CIP-0031 , https://cips.cardano.org/cip/CIP-0032 , https://cips.cardano.org/cip/CIP-0033
- [S34] IOG, "Unlocking more opportunities with Plutus V3": https://www.iog.io/news/unlocking-more-opportunities-with-plutus-v3 ; Chang upgrade docs: https://docs.cardano.org/about-cardano/evolution/upgrades/chang/ ; Cexplorer: https://cexplorer.io/article/chang-hard-fork-brings-plutus-v3-to-cardano
- [S35] SundaeSwap scalability (concurrency, scoopers): https://sundae.fi/posts/sundaeswap-scalability
- [S36] Minswap batcher docs: https://docs.minswap.org/courses/how-to-perform-swaps/batcher ; IOG, "Architecting dApps on the eUTxO ledger": https://www.iog.io/news/architecting-dapps-on-the-eutxo-ledger
- [S37] Order book pattern: https://plutus-apps.readthedocs.io/en/latest/plutus/explanations/order-book-pattern.html ; Genius Yield launch: https://cardanospot.beehiiv.com/p/genius-yield-orderbook-dex-mainnet-launch
- [S38] Developer Portal design patterns (stake validator, UTxO indexers): https://developers.cardano.org/docs/build/smart-contracts/advanced/design-patterns/overview/ , https://developers.cardano.org/docs/build/smart-contracts/advanced/design-patterns/stake-validator , https://developers.cardano.org/docs/build/smart-contracts/advanced/design-patterns/utxo-indexers
- [S39] CIP-118 Nested Transactions: https://cips.cardano.org/cip/CIP-0118 ; Babel fees: https://www.iog.io/news/babel-fees ; TSC review: https://technicalsteeringcommittee.docs.intersectmbo.org/reviews/tsc-review-on-nested-transactions-inquiry
- [S40] CIP-113 programmable tokens: Cardano Foundation Feb 2026 update https://cardanofoundation.org/blog/february-2026-activities ; CertiK https://www-cn.certik.com/blog/a-long-overdue-innovation-in-cardano-interoperable-programmable-token-design ; Minswap/FluidTokens proposal https://www.lidonation.com/en/proposals/minswap-fluid-tokens-a-smart-token-dex-on-cardano-f12 ; USDCx mention https://blockchain.news/flashnews/cardano-s-progress-usdcx-launch-defi-growth-and-programmable-tokens
- [S41] Cardano.org glossary, Amaru: https://cardano.org/glossary/amaru/ ; CoinTurk: https://en.coin-turk.com/cardano-dijkstra-hard-fork-on-schedule-amaru-node-targets-november-2026/
- [S42] Intersect, Plomin ratified: https://www.intersectmbo.org/news/plomin-hard-fork-ratified ; Plutus ledger API protocol versions: https://plutus.cardano.intersectmbo.org/haddock/latest/plutus-ledger-api/PlutusLedgerApi-Common-ProtocolVersions.html ; CIP-122 https://cips.cardano.org/cip/CIP-0122 ; CIP-127 https://cips.cardano.org/cip/CIP-0127
- [S43] GitHub releases API (queried 2026-10-07): aiken-lang/aiken, IntersectMBO/plutus, Plutonomicon/plutarch-plutus, scalus3/scalus, OpShin/opshin, Python-Cardano/pycardano, bloxbean/cardano-client-lib, geniusyield/atlas, cardano-scaling/hydra, txpipe/dolos, txpipe/oura, bloxbean/yaci-devkit, bloxbean/yaci-store, CardanoSolutions/ogmios, CardanoSolutions/kupo, IntersectMBO/cardano-node, IntersectMBO/cardano-db-sync, blockfrost/blockfrost-backend-ryo, cardano-foundation/cardano-ibc-incubator
- [S44] IOG, "Plutus Tx gets a makeover: meet Plinth": https://www.iog.io/news/plutus-tx-gets-a-makeover-meet-plinth
- [S45] Scalus: https://scalus.org ; 2026 proposal https://hackmd.io/@lantr/scalus2026 ; Scala index https://index.scala-lang.org/scalus3/scalus ; emulator office hours https://forum.cardano.org/t/developer-office-hours-55-scalus-emulator-in-memory-cardano-node-for-jvm-js-ts-testing/153852
- [S46] Helios: https://helios-lang.io ; https://github.com/HeliosLang/compiler
- [S47] Tx3: https://developers.cardano.org/tools/tx3/ ; https://docs.txpipe.io/tx3 ; office hours https://developers.cardano.org/blog/2025-10-17-media-cardano-developer-office-hours/
- [S48] tx3.land: https://tx3.land
- [S49] Aiken Hello World: https://aiken-lang.org/example--hello-world/basics ; playground https://play.aiken-lang.org
- [S50] Mesh site, AI page and course: https://meshjs.dev , https://meshjs.dev/ai , https://meshjs.dev/resources/cardano-course , https://mimir.meshjs.dev
- [S51] UTxO RPC: https://utxorpc.org
- [S52] Blockfrost docs: https://blockfrost.dev/start-building
- [S53] Blockfrost free tier 50k/day (secondary): https://developers.cardano.org/docs/get-started/infrastructure/api-providers/blockfrost/overview , https://us.fitgap.com/products/blockfrost
- [S54] Koios: https://koios.rest
- [S55] Maestro product updates (Bitcoin and BitcoinOS focus): https://maestro-product.beehiiv.com/p/maestro-strategic-partnerships-bitcoinos-and-midnight
- [S56] Ogmios CHANGELOG (v7.0.0): https://github.com/CardanoSolutions/ogmios/blob/master/CHANGELOG.md
- [S57] Dolos README: https://github.com/txpipe/dolos ; docs https://docs.txpipe.io/dolos
- [S58] Mithril: https://mithril.network/doc ; https://developers.cardano.org/docs/operators/operator-tools/mithril/ ; repo https://github.com/IntersectMBO/mithril
- [S59] Yaci DevKit: https://devkit.yaci.xyz
- [S60] Testnet environments (Preview vs Preprod): https://book.world.dev.cardano.org/env-preview.html , https://book.world.dev.cardano.org/env-preprod.html , https://docs.cardano.org/cardano-testnets/environments
- [S61] Cardano.org glossary, SanchoNet: https://cardano.org/glossary/sanchonet/ ; https://sancho.network
- [S62] Official faucet: https://docs.cardano.org/cardano-testnets/tools/faucet
- [S63] Faucet amount and rate (secondary): https://www.datawallet.com/de/krypto/get-cardano-testnet-tokens
- [S64] Intersect Plutus Core update (UPLC optimizer/certifier results): https://updates.cardano.intersectmbo.org/2026-04-22-plutus-core
- [S65] Intersect DevEx, Cardano Developer Pathway resources: https://devex.intersectmbo.org/docs/working-group/sessions/q1-2026/cardano-developer-pathway/session-resources
- [S66] Cardano Foundation developer-portal issue "Ecosystem DevEx 2026": https://github.com/cardano-foundation/developer-portal/issues/1759 ; "Portal 2026": https://github.com/cardano-foundation/developer-portal/issues/1758
- [S67] Intersect DevEx FAQ: https://devex.intersectmbo.org/docs/faq
