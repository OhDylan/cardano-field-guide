# 03 — AI Agents on Cardano, and Midnight & Partner Chains

*Research for a builder-focused site, aimed at engineers coming from Ethereum/Solana, web2, or AI. Compiled 2026-10-07. Every non-obvious claim has a source link (numbers like [S12] point to the **Sources** list at the end). Anything I could not confirm is marked **[UNVERIFIED]**. Where sources disagree, both versions are given.*

---

## TL;DR for builders

| Area | What's live today (Oct 2026) | What's still coming | Fastest "try it" |
|---|---|---|---|
| **Masumi** (agent payments, identity and registry) | Escrow payment and registry smart contracts on Cardano **Mainnet and Preprod**. Python SDK `masumi` v1.2.0. Self-hosted node (Docker) or hosted "Masumi as a Service". Sokosumi marketplace. MCP servers. x402 support. [S1][S3][S5][S7] | Masumi L2 (Hydra or rollup) for cheap micropayments; `$SUMI` governance token (not launched) [S6][S10] | `pip install masumi && masumi init`, then run the Docker quickstart node on Preprod (steps in §A5) |
| **x402 on Cardano** | Cardano "exact" scheme merged into the x402 spec. `@x402/cardano` on npm (v2.28.0). Cardano Foundation facilitator (Java). Masumi "vending machine" demo [S11][S12][S13][S14] | The CF facilitator has been proven only on Preprod and is not yet run on mainnet [S13][S14] | `npm i @x402/cardano`, or try https://www.masumi.network/vending-machine |
| **Cardano MCP / AI dev tooling** | Indigo `@indigoprotocol/cardano-mcp`, Mesh MCP and skills, Cardano Foundation `cardano-dev-skills` plugin, Masumi and Sokosumi MCP, Midnight Kapa MCP and Midnight Expert [S15][S16][S17][S18][S33] | An Intersect "production-grade Cardano MCP" effort is in progress [S19] | `claude mcp add --transport http midnight https://midnight.mcp.kapa.ai` |
| **Midnight** (ZK privacy partner chain) | **Mainnet live** (genesis block 2026-03-17) with federated node operators. **Permissionless contract deployment since about 2026-10-02.** Blockfrost is the public mainnet RPC and indexer. Wallets: Lace, 1AM, Gero, CTRL [S20][S21][S22][S23][S24] | Mōhalu: Cardano SPOs as block producers, DUST Capacity Exchange, incentives. Hua: full decentralization and "hybrid dApps" on other chains [S25][S26] | `npx create-mn-app@latest my-app && cd my-app && npm run setup` (local devnet, no faucet needed) [S27] |
| **Partner Chains toolkit** (Substrate) | Code now lives inside `midnightntwrk/midnight-node`. The standalone IOG repo was **archived 2026-04-23** [S28] | Third-party partner chains are still early: Charli3 oracle chain, Materios, Nuvola [S29][S30] | Read the archived repo docs; build from the midnight-node repo |

---

# Part A — AI agents and agentic protocols on Cardano

## A1. Masumi Network: who builds it and what it is

- **Builders:** Masumi was co-founded by **NMKR**, a Cardano NFT and payments infrastructure company led by Patrick Tobler, and **Serviceplan Group**, a European marketing and communications group, through its Plan.Net arm. It launched in **November 2024** [S2][S8]. The Cardano Foundation published a case study on it [S9]. Its Catalyst Fund 14 proposal, "Masumi AI Agent Network 2.0 by Serviceplan Group", asked for **900k ADA**. About 561k ADA has been distributed so far, for a 10-month project running Feb–Oct 2026. Milestones done: Identity framework and Developer SDK. In progress: L2 architecture and prototype, about 50% complete [S10].
- **Pitch:** "Masumi is the payment network for AI agents. Escrow smart contracts, on-chain identity, and a public registry let autonomous agents transact without trusting each other" (masumi.network meta description) [S1].
- **Framework-agnostic.** Docs and templates cover CrewAI, AutoGen, LangGraph and LangChain, PhiData/Agno, n8n, and MCP [S3][S31].
- **Positioning:** Masumi targets enterprises and EU compliance, citing MiCA, GDPR and the AI Act [S2]. It is closer to a "Stripe plus escrow plus registry for agents" than to a token launchpad.

### Architecture

| Component | What it does | Notes |
|---|---|---|
| **Masumi Node: Payment Service** | TypeScript service with Postgres. It manages three wallets (Purchasing, Selling, Collection), builds and submits escrow transactions, and exposes a REST API (`/payment`, `/purchase`, `/registry`, `/wallet`, and refund endpoints) plus an admin dashboard at `:3001/admin` | Repo: `masumi-network/masumi-payment-service` [S3][S4] |
| **Registry Service** | Indexes on-chain agent registrations and exposes search and discovery: `/registry-entry-search`, `/payment-information`, `/capability` | Optional to self-host. A central instance is at `registry.masumi.network` [S3] |
| **Registry smart contract (identity)** | Each agent is an **NFT minted on Cardano** whose metadata follows MIP-002. To update an agent, you deregister it (burning the NFT) and register again | [S3] |
| **Payment smart contract (escrow)** | The buyer locks funds. The seller submits a result hash. A dispute window runs until `unlockTime`. Then funds are released, or refunded on request | States are visible in `NextAction.requestedAction` (`FundsLockingRequested`, then `FundsLocked`, then `ResultSubmitted`, then `Completed`) [S3] |
| **Decision logging** | A SHA-256 hash of input and output (MIP-004) is written on-chain. It is required to unlock payment and is the evidence in disputes. Only hashes go on-chain, never the data | [S3] |
| **MIP standards** | MIP-001 (process), **MIP-002** (on-chain registry metadata), **MIP-003** (Agentic Service API: `/start_job`, `/status`, `/availability`, `/input_schema`, plus `/provide_input` for human-in-the-loop), MIP-003 Att. 01 (input schema format), **MIP-004** (input and output hashing) | [S3] |
| **Kodosumi** | Open-source (Apache-2.0) Python runtime on **Ray Serve**, with an admin panel, `koco` CLI, and Masumi payment integration for running agent workflows at scale | `pip install kodosumi` [S32] |
| **Sokosumi** | "Fiverr for AI agents" marketplace (web2 UX, credits). Hosted MCP at `https://mcp.sokosumi.com/mcp` (OAuth). Preprod gallery at `preprod.sokosumi.com/agents`. Mainnet listing requires whitelisting | [S3][S34] |
| **Masumi as a Service** | Hosted dashboard at `app.masumi.network`: KYC/KYB, org API keys, OIDC, agent registration without running your own node | [S3] |

### Fees and tokens (important for pricing)

- **Network fee:** Masumi charges **5% of the selling price** in the active stablecoin. Cardano transaction fees in ADA are paid for locking, submitting results, collecting, and registering or deregistering [S6].
- **Stablecoin: the docs are inconsistent here.** The current **Tokens** page says mainnet uses **USDCx** (Circle xReserve on Cardano, policy `1f3aec8b…7e34`, asset `USDCx`) and Preprod uses **tUSDM**. The registration and Sokosumi pages still say to price in **USDM** for Sokosumi listing [S6][S3]. Tell readers to use tUSDM on Preprod and to check the Tokens page before going to mainnet.
- **No `$SUMI` token yet.** It is planned as a later governance token [S6].
- Running directly on L1 makes very small payments uneconomic. Masumi's L2 is meant to fix this [S6][S10].

### Traction (as published by NMKR, covering Jan–Oct 2025)

These figures come from NMKR's client page: 16,900+ transactions on mainnet and testnet, **$23k+ verified mainnet volume**, 250+ live agents on Sokosumi, 4,461 jobs, 5,830+ users (2,158 paying), about $4k platform revenue at a 17.5% take rate, and BMW and Generali named as enterprise users [S8]. **2026 figures: [UNVERIFIED]**. I found no updated public stats.

## A2. Two payment rails: MIP-003 escrow vs x402

| | **MIP-003 (Masumi escrow)** | **x402 (HTTP 402)** |
|---|---|---|
| Flow | `start_job`, then lock funds in the escrow contract, then the agent works, then the result hash goes on-chain, then the dispute window, then payout | The client calls the API, gets a 402 with `PaymentRequirements`, retries with a signed Cardano transaction in the `PAYMENT-SIGNATURE` header, and the server or facilitator verifies and broadcasts it |
| Refunds and disputes | Yes | No, payment is final (about 20 s settlement), *unless* you use `assetTransferMethod: "masumi"`, which routes through the escrow contract [S7] |
| Best for | High-value jobs where the buyer needs assurance | Metered API calls and micropayments |

**x402 on Cardano timeline.** Masumi first showed x402 on Cardano with a "memecoin mint" demo paid in USDM in October 2025 [S35]. The **Cardano exact scheme** spec (`specs/schemes/exact/scheme_exact_cardano.md`) uses network IDs `cardano:mainnet`, `cardano:preprod` and `cardano:preview`. It supports ADA or any single native token (`policyId.assetNameHex`), uses a UTxO nonce as a replay guard, and has a Masumi-escrow variant [S12]. The merge into the official x402 repo was reported in April 2026 by some outlets and around 2026-09-09 by others. On **2026-09-21**, Cardano was added to the official x402 SDK as `@x402/cardano` [S11][S36]. npm shows `@x402/cardano` v2.28.0, published 2026-09-29 [npm]. The Cardano Foundation's Java facilitator (`cardano-foundation/cardano-x402-facilitator`) supports `default`, `masumi` and `script` transfer methods. **It has been proven on Preprod only, not on mainnet** [S13][S14].

## A3. Other agent infrastructure on Cardano

| Tool | What | Link / install |
|---|---|---|
| **Indigo `cardano-mcp`** | Wallet-aware MCP server: addresses, UTxOs, balances, ADA Handles, staking, submitting signed transactions. Uses Blockfrost, Kupo or Ogmios. The seed is stored locally | `npx @indigoprotocol/cardano-mcp setup` (v1.0.11) [S15] |
| **Masumi MCP server** | Lets Claude Desktop and similar clients list, hire and monitor Masumi agents (`list_agents`, `hire_agent`, `check_job_status`). Requires your own Masumi node | github.com/masumi-network/masumi-mcp-server [S3] |
| **Sokosumi MCP** | Hosted, OAuth. Lets you browse agents, create jobs and check credits | `https://mcp.sokosumi.com/mcp` [S34] |
| **Masumi Skills** | Coding-assistant skill for Claude Code, Cursor and others | masumi.network/skill.md, github.com/masumi-network/masumi-skills [S3] |
| **Cardano Foundation `cardano-dev-skills`** | Claude Code plugin and skill pack with docs refreshed weekly | `/plugin marketplace add cardano-foundation/cardano-dev-skills` [S17] |
| **Mesh SDK AI tools** | Mesh MCP, agent skills, llms.txt | meshjs.dev/ai [S16] |
| **Unified / community Cardano MCPs** | e.g., easy1staking "Cardano Unified MCP" (Blockfrost, Koios, Maestro; 38 tools) | glama.ai listings [S19] |
| **Intersect DevEx WG** | Q2 2026 sessions on "Building a production-grade MCP server for Cardano" and "AI in your Cardano dev workflow" | [S19] |
| **developers.cardano.org: AI Agents curriculum** | Official guidance. Agents use ordinary SDK wallets. "The agent proposes and you sign". Treat any MCP server that can move funds as custodial | [S31] |

**Agent wallets.** No Cardano-specific "agent wallet with spending policy" standard (a CIP) was found: **[UNVERIFIED / appears absent]**. In practice, agent keys are either the Masumi node's managed hot wallets (Purchasing and Selling wallets plus a cold Collection address) [S3], or SDK wallets built with Mesh, Lucid Evolution or Blaze, with native scripts or Aiken validators enforcing limits. This gap could be a good opportunity for builders.

**Cardano Foundation and IOG positions.** Cardano Foundation leadership has talked publicly about agent accountability and delegated authority in 2026 [S37]. Cardano Foundation engineers built the x402 Cardano client, server and facilitator [S11][S13]. Hoskinson (IOG) said in June 2026 that AI agents are core infrastructure for the Midnight ecosystem. **Midnight City** is a live simulation of autonomous AI agents generating ZK transactions [S38][S39]. Reports of an IOG and Masumi "deploy Masumi on Hydra" collaboration in Feb 2026 are **[UNVERIFIED]**: secondary sources only. Masumi's own L2 milestone mentions Hydra or rollups [S10].

**Olas- or Virtuals-style tokenized agent launchpads on Cardano:** I found no significant live equivalent **[UNVERIFIED as absent]**. The ecosystem has bet on payments, identity and registry infrastructure (Masumi, x402) rather than agent tokens.

**Catalyst:** Agent-related proposals appear across Funds 13–15. Examples: Masumi 2.0 (F14, funded), the Aegis AI security guardian (F14 proposal), and LLM analytics proposals in F15 [S10][S40]. Funding status beyond Masumi is **[UNVERIFIED]**.

## A4. Masumi SDK reference

**Python (`pip install masumi`, v1.2.0, released 2026-02-13)** [S5]:

```python
from masumi import run

async def process_job(identifier_from_purchaser: str, input_data: dict):
    return input_data.get("text", "").upper()   # return a string

INPUT_SCHEMA = {"input_data": [{"id": "text", "type": "string", "name": "Text"}]}

if __name__ == "__main__":
    run(start_job_handler=process_job, input_schema_handler=INPUT_SCHEMA)
    # env: PAYMENT_API_KEY, SELLER_VKEY, AGENT_IDENTIFIER, PAYMENT_SERVICE_URL, NETWORK=Preprod
```

CLI commands: `masumi init`, `masumi check`, `masumi run [file] [--standalone --input '{...}']`. Running the agent exposes a FastAPI server with all MIP-003 endpoints at `localhost:8080/docs`. Lower-level classes are `create_masumi_app`, `Payment` and `Purchase`. Human-in-the-loop support uses `request_input()` and `/provide_input` [S5][S3].

**TypeScript.** The Payment and Registry services are TypeScript with published OpenAPI specs, so you can generate a client. A dedicated "official TS agent SDK" package is **[UNVERIFIED]**. The F14 milestone lists "SDK in at least one major language" as done [S10]. Masumi Skills include TS examples [S3].

**Templates:** `crewai-masumi-quickstart-template`, `agno-masumi-reference-implementations`, an n8n community node, and a Railway deploy template [S4][S3].

## A5. "Hello world": deploy and register a paid agent on Masumi Preprod in an afternoon

*Prerequisites: Docker, Python 3.10+, a free Blockfrost **Preprod** project key (blockfrost.io), and ngrok or a public host if the node and agent run on different machines.*

1. **Run a Masumi node locally.** Use the Docker Compose quickstart [S4]:
   ```bash
   git clone https://github.com/masumi-network/masumi-services-dev-quickstart.git
   cd masumi-services-dev-quickstart && cp .env.example .env
   # set ADMIN_KEY (>=15 chars), ENCRYPTION_KEY (>=20 chars), BLOCKFROST_API_KEY_PREPROD
   docker compose up -d
   ```
   This starts the Registry at `:3000/docs`, the Payment Service at `:3001/docs`, the admin UI at `:3001/admin`, and Postgres at `:5432`. *If you would rather not self-host, use app.masumi.network ("Masumi as a Service") [S3].*
2. **Back up the generated wallet mnemonics** from the admin UI, then **fund the Selling and Purchase wallets with test ADA** from the Cardano Preprod faucet (https://docs.cardano.org/cardano-testnets/tools/faucet) or the Masumi faucet linked in the docs [S3].
3. **Create the agent:**
   ```bash
   python3 -m venv venv && source venv/bin/activate
   pip install masumi && masumi init
   ```
   Edit `process_job` and call your LLM, CrewAI crew or LangGraph graph inside it [S5].
4. **Configure `.env`:** copy `PAYMENT_API_KEY` from Admin → API Keys and `SELLER_VKEY` from Admin → Wallets → Selling wallet. Set `NETWORK=Preprod` and `PAYMENT_SERVICE_URL=http://localhost:3001/api/v1`. Then run `masumi check` [S3][S5].
5. **Run the agent** with `masumi run`. Expose it publicly with ngrok if needed.
6. **Register it** in Admin → AI Agents → **+ Register AI Agent**. Fill in name, description, API URL, capability, tags and author, and set the price in **tUSDM**. This mints the registry NFT. Wait 5–15 minutes on Preprod, then copy `agentIdentifier` into `.env` as `AGENT_IDENTIFIER` and restart [S3].
7. **Verify** with `POST /registry-entry-search` on the Registry, in the Masumi Explorer, or at `https://preprod.sokosumi.com/agents`. A tUSDM-priced Preprod agent appears there automatically [S3].
8. **Test a purchase end to end:**
   - Call `POST /start_job` on your agent.
   - Call `POST /purchase` on the node with the `blockchainIdentifier`, the timing fields, and the SHA-256 `inputHash`.
   - Poll `GET /purchase` until it reaches `FundsLocked`, then `ResultSubmitted`.
   - Fetch the result from `GET /status`.
   Steps are in "Payments & Escrow" [S3]. Alternatively, hire the agent from Claude through the Masumi MCP server.
9. **Optional:** add an x402 endpoint using `@x402/cardano` for per-call pricing [S11].
10. **Going to mainnet:** run a separate node, fund the wallets with ADA plus the stablecoin (USDCx per the Tokens page), and submit the Sokosumi whitelisting form [S3][S6].

---

# Part B — Midnight and partner chains

## B1. What Midnight is

Midnight is a **privacy ("data-protection") blockchain that runs as a Cardano partner chain**. It uses zk-SNARKs for **selective disclosure**: you prove facts without revealing the underlying data. It keeps **public (unshielded) and private (shielded) state side by side**. It is built on Substrate via the Partner Chains stack. Developed by Shielded Technologies (an IOG spin-out) and stewarded by the **Midnight Foundation** [S20][S41].

### Status timeline

| Date | Event | Source |
|---|---|---|
| 2025-08-05 → 10-20 | **Glacier Drop**: claims open to self-custody holders of ADA, BTC, ETH, SOL, XRP, BNB, AVAX and BAT. **3.547B NIGHT to 170k+ wallets** | [S42][S43] |
| 2025-10-30 → 11-19 | **Scavenger Mine**: compute-puzzle phase open to anyone. **1B NIGHT to 8M+ wallets** | [S42][S44] |
| 2025-12-04 | **NIGHT launched as a Cardano native asset** (Hilo phase). Redemption runs over 360 days in 4×25% thaws. **Lost-and-Found** (252M NIGHT) stays open for about 4 years after mainnet | [S42] |
| 2026-02 | Consensus HK: mainnet set for end of March. **LayerZero** integration announced | [S45] |
| **2026-03-17** | **Genesis block (Kūkolu, federated mainnet).** Launch post dated 2026-03-29. *Some outlets say "went live March 31"* | [S20][S21][S38] |
| 2026-03 | Federated node operators: Google Cloud, Blockdaemon, MoneyGram, Worldpay, Bullish, Pairpoint by Vodafone, eToro, AlphaTON Capital, Shielded Technologies (the blog says "twelve") | [S20][S21] |
| 2026-03 → 09 | **Gated deployment**: mainnet contracts needed a Foundation security review after Preprod (a rubric plus a PR to the MIP repo) | [S22][S46] |
| 2026-08 | Node 2.1.0-beta (live-chain upgrades, ledger 8→9). Sundae Labs **Capacity Exchange** pays fees in **USDM** instead of DUST. VIA Labs cross-chain messaging. Gero and 1AM wallets add DUST generation | [S26] |
| 2026-09-26 | Node 1.0.3, described as the "final pre-smart-contract release" | [S47] |
| 2026-09-30 | Midnight-hosted mainnet endpoints retired. **Blockfrost** is now the public mainnet RPC and indexer | [S48] |
| **~2026-10-02** | **Permissionless smart-contract deployment on mainnet.** No mandatory review. Contracts can deploy contracts, and apps can deploy contracts for users. *Reported by multiple news outlets. I did not find an official Midnight blog post* | [S22][S23][S47] |
| Next | **Mōhalu** (planned Q2–Q3 2026, apparently not yet started): Cardano SPOs onboarded as block producers, incentives, DUST Capacity Exchange, first on-chain governance. **Hua**: full decentralization, "hybrid dApps" that embed Midnight privacy in apps on other chains | [S25][S26][S49] |

### NIGHT and DUST: the token model, explained for EVM developers

| | **NIGHT** | **DUST** |
|---|---|---|
| Type | Unshielded native token. **24B fixed supply** | Shielded, **non-transferable**, decaying *resource* (not a token you can trade) |
| Role | Governance, block rewards, treasury, **generating DUST** | Pays transaction fees |
| Mechanics | Hold NIGHT and **register or designate** it to a DUST address | Accrues like a battery up to a cap (about 5 DUST per NIGHT, roughly 7 days to full). Decays to zero if the backing NIGHT moves [S26][S50] |
| Cross-chain | **cNIGHT** lives on Cardano and can generate DUST through the cNIGHT observation pallet, with about 12 h of Cardano finality. **mNIGHT** lives on Midnight. The MIP-20 "switch" bridge is **one-way, Cardano→Midnight**, for now [S50][S51][S52] | Supports **fee sponsorship**: dApps can pay users' fees (Ura Finance testing, Gero support) [S23][S26] |

The key idea for developers: **fees are predictable and not tied to token price**, and apps can **sponsor DUST** so users never need to hold a token. The Capacity Exchange lets users pay in other assets such as USDM [S26][S41].

### How Midnight relates to Cardano

- **Security and SPOs.** Midnight is a Partner Chain. Its validator set mixes permissioned nodes with **registered Cardano SPOs**, weighted by a governance parameter `D`. Today only Federated Node Operators produce blocks. **SPO onboarding is "supported at a later date"** (Mōhalu) [S24][S49]. Testnet-02 had 180+ SPOs participating [S49].
- **Liquidity.** NIGHT originated as a Cardano native asset, so cNIGHT trades on Cardano DEXes and in Cardano wallets (Eternl, NuFi, Lace, hardware wallets) [S51]. Cardano holders can generate DUST without bridging [S50].
- **Cross-chain.** LayerZero integration was announced [S45], VIA Labs messaging went live [S26], and Ascend perps settle on Cardano, EVM and Solana [S26]. Paying fees from other chains is the Capacity Exchange plus hybrid dApps roadmap (Mōhalu and Hua) [S25][S26].

## B2. Developer stack

| Piece | Current | Notes |
|---|---|---|
| **Compact** | Compiler **0.31.1**. `pragma language_version 0.23`. Runtime 0.16.0 per the support matrix; `@midnight-ntwrk/compact-runtime` 0.20.0 was published to npm 2026-09-29 | TypeScript-like DSL that compiles to ZK circuits **and** a TypeScript API [S53][S54] |
| Install | `curl --proto '=https' --tlsv1.2 -LsSf https://github.com/midnightntwrk/compact/releases/latest/download/compact-installer.sh \| sh` then `compact update 0.31.1` | Linux and macOS. Windows via WSL only [S53] |
| **Proof server** | `docker run -p 6300:6300 midnightntwrk/proof-server:8.1.0 midnight-proof-server -v` | Runs locally, so private inputs never leave the machine [S53] |
| **Midnight.js** | 4.1.1. Wallet SDK 1.2.0. DApp Connector API 4.0.1. Indexer 4.3.x (GraphQL) | [S54] |
| **Scaffolder** | `npx create-mn-app@latest my-app` (v0.5.1, midnightntwrk org). Templates: hello-world, bboard, battleship, leaderboard. Bundled local devnet with pre-minted NIGHT. `--network preview\|preprod` | [S27] |
| **Networks** | **Preview** (early dev), **Preprod** (final staging), **Mainnet** (Blockfrost) | RPC: `rpc.preview.midnight.network`, `rpc.preprod.midnight.network`, `rpc.midnight-mainnet.blockfrost.io` [S48] |
| **Faucets** | tNIGHT: Preprod at https://midnight-tmnight-preprod.nethermind.dev/, Preview at https://midnight-tmnight-preview.nethermind.dev/ (1,000 tNIGHT per request). Then press "Generate tDUST" in Lace. A Google Cloud faucet was also reported | [S55][S23] |
| **Wallets** | Lace (Midnight), 1AM, Gero (v2.7+, shielded), CTRL | [S21][S26][S23] |
| **AI tooling** | Kapa MCP: `claude mcp add --transport http midnight https://midnight.mcp.kapa.ai`. **Midnight Expert** Claude Code plugins: `curl -fsSL https://midnightntwrk.expert/install.sh \| bash`. The older npm `midnight-mcp` is **deprecated** | [S33][S56] |
| **Learning** | Midnight Academy (with HackQuest), Developer Fireside every Wednesday at 15:00 UTC, Dev Diaries | [S21][S57] |
| **Funding** | Night Sky Accelerator (10 weeks, from Jul 2026). MLH Midnight hackathons (May and Jul 2026). AKINDO Buildathon ($12.5k, Japan). Aliit Fellowship | [S57][S26][S45] |
| **Data** | Dune and Token Terminal dashboards. Explorers: midnightexplorer.com, Subscan | [S58][S46] |

### Compact example (verbatim from `midnightntwrk/example-bboard`) [S59]

```compact
pragma language_version 0.23;
import CompactStandardLibrary;

export enum State { VACANT, OCCUPIED }
export ledger state: State;                       // public on-chain state
export ledger message: Maybe<Opaque<"string">>;
export ledger sequence: Counter;
export ledger owner: Bytes<32>;

constructor() {
  state = State.VACANT;
  message = none<Opaque<"string">>();
  sequence.increment(1);
}

witness localSecretKey(): Bytes<32>;              // private input, supplied by TypeScript, never on-chain

export circuit post(newMessage: Opaque<"string">): [] {
  assert(state == State.VACANT, "Attempted to post to an occupied board");
  owner = disclose(publicKey(localSecretKey(), sequence as Field as Bytes<32>));
  message = disclose(some<Opaque<"string">>(newMessage));
  state = State.OCCUPIED;
}

export circuit takeDown(): Opaque<"string"> {
  assert(state == State.OCCUPIED, "Attempted to take down post from an empty board");
  assert(owner == publicKey(localSecretKey(), sequence as Field as Bytes<32>), "Not the current owner");
  const formerMsg = message.value;
  state = State.VACANT;
  sequence.increment(1);
  message = none<Opaque<"string">>();
  return formerMsg;
}

export circuit publicKey(sk: Bytes<32>, sequence: Bytes<32>): Bytes<32> {
  return persistentHash<Vector<3, Bytes<32>>>([pad(32, "bboard:pk:"), sequence, sk]);
}
```

**Mental model for EVM and Solana developers:**

- `ledger` is like contract storage and is public.
- `circuit` is like an external function. The compiler turns it into a ZK circuit, and the caller proves locally that they executed it correctly.
- `witness` is a private input supplied by the dApp's TypeScript, for example from local storage.
- `disclose()` must explicitly mark any private-derived value that becomes public. The compiler rejects accidental leaks.

In the example, ownership is proven by knowing a secret key whose hash matches `owner`, and the key is never revealed. The minimal hello-world is just `export ledger message: Opaque<"string">; export circuit storeMessage(m: Opaque<"string">): [] { message = disclose(m); }` [S60].

### What Midnight is good for

| Use case | Why Midnight | Live or example |
|---|---|---|
| **Private identity and credentials** | Prove age, residency, KYC status or membership without revealing the data | ZK Identity challenge [S61] |
| **Compliant DeFi and stablecoins** | Shielded balances plus selective disclosure to auditors and regulators | W3i **shieldUSD** [S61]; Ascend perps [S26]; Zswap atomic swaps with Celestia [S26] |
| **Voting and governance** | Prove eligibility and cast a vote without linking it to identity | Common tutorial pattern **[example concept]** |
| **Healthcare and data sharing** | Prove facts about records (for example "vaccinated" or "in trial cohort") while the data stays local | Hackathon theme **[example concept]** |
| **Games with hidden info** | Hidden state proven fair | Battleship and Sea Battle templates [S27][S61]; **Dominion** on-chain poker (Webisoft) [S21] |
| **Enterprise traceability** | Private supply-chain proofs | Petrobras traceability apps (reported) [S22] |
| **AI agents with private state** | Agents act on private data with provable behaviour | **Midnight City** simulation (V2 live) [S39][S58] |

## B3. Partner Chains SDK (Substrate)

- **What it is:** IOG's Rust toolkit, built on the Polkadot SDK, that lets a Substrate chain **borrow Cardano security**. Chain builders get pallets and Cardano smart contracts for:
  - committee selection mixing **permissioned nodes and registered Cardano SPOs**, with the ratio set by the **D-parameter** (the "Ariadne", later "Minotaur", selection algorithm);
  - multisig governance authorities;
  - **native token reserve management** observed from Cardano;
  - block-production rewards for producers and delegators [S28][S62].
- **How SPOs secure partner chains:** An SPO registers on Cardano, signing with its pool cold key plus the partner-chain keys. The partner chain observes Cardano stake distribution and picks committee members weighted by stake. SPOs run the partner-chain node next to their Cardano node and earn rewards in the partner chain's token [S28][S62][S24].
- **Status (important):** The `input-output-hk/partner-chains` repo was **archived 2026-04-23**: "Partner chains is now directly part of Midnight". Development continues in `midnightntwrk/midnight-node` [S28]. Partner Chains v1.8.0 shipped with Midnight node 0.18 in Dec 2025 [S63].
- **Who uses it:** **Midnight** is the first and only one in production. **Charli3** forked the toolkit for an oracle partner chain (MVP stage) [S29]. **Materios** and **Nuvola/Vola** (DePIN) are described as building partner chains [S30]. Their production status is **[UNVERIFIED]**. **Apex Fusion** (VECTOR and NEXUS) is "Cardano-aligned", but it is a Cardano-codebase fork plus EVM, **not** the Substrate toolkit [S64].
- **Builder takeaway:** Treat the partner-chain toolkit as **Midnight-centric infrastructure** for now. Launching your own partner chain is possible but niche. Most developers should deploy *on* Midnight rather than *build* a new partner chain.

---

## Ideas for interactive website content (this area)

1. **"Hire an agent from your browser" live demo.** A widget that calls the **x402 vending machine** or a demo Masumi agent on Preprod. It shows the HTTP 402 → signed transaction → 200 handshake step by step, with each request and response JSON and the Preprod transaction on an explorer.
2. **Escrow lifecycle visualizer.** An animated state machine (`FundsLocked → ResultSubmitted → dispute window → Completed/Refunded`) with sliders for `submitResultTime` and `unlockTime`, a "tamper with output" button that breaks the MIP-004 hash, and a side-by-side table comparing MIP-003 with x402.
3. **Compact playground with a privacy lens.** An editor showing the bboard contract. Hovering over `ledger`, `witness` or `disclose()` highlights what is public vs private, and a "remove disclose()" toggle shows the compiler error. Link out to `npx create-mn-app`.
4. **NIGHT → DUST battery calculator.** Enter NIGHT held and transactions per day to see DUST generation (cap and recharge), decay when NIGHT moves, and how much sponsorship a dApp needs for N users. Compare it with "gas on Ethereum at X gwei".
5. **"Pick your agent stack" decision tree.** Answers to questions like "do you need refunds?", "do you need a marketplace?" and "do you want to self-host?" route to Masumi node, Masumi-as-a-Service, x402 only, Sokosumi listing, or MCP-only. Each route ends in copy-paste setup commands, with a "Live vs Coming" status badge on every component.

---

## Sources

- [S1] Masumi homepage and dev hub — https://www.masumi.network/ , https://www.masumi.network/dev
- [S2] Masumi launch coverage: AI News — https://www.artificialintelligence-news.com/news/masumi-network-how-ai-blockchain-fusion-adds-trust-to-burgeoning-agent-economy/ ; Serviceplan/Plan.Net — https://house-of-communication.com/de/en/brands/plan-net/landingpages/agentic-services/masumi.html ; CF partnership — https://cardanofoundation.org/blog/serviceplan-cardano-foundation-partnership
- [S3] Masumi docs (llms index generated 2026-10-06) — https://www.masumi.network/dev/llms.txt ; pages: core-concepts/payments, decision-logging, environments, wallets; get-started/register-agent, install-masumi-node, masumi-as-a-service; mips/_mip-003; technical-documentation/_masumi-mcp-server; integrations/masumi-skills; how-to-guides/list-agent-on-sokosumi (all under https://www.masumi.network/dev/masumi/…)
- [S4] Masumi dev quickstart README — https://github.com/masumi-network/masumi-services-dev-quickstart ; org — https://github.com/masumi-network
- [S5] `masumi` on PyPI (v1.2.0, 2026-02-13) — https://pypi.org/project/masumi/
- [S6] Masumi Tokens and Transaction Fees — https://www.masumi.network/dev/masumi/core-concepts/tokens , https://www.masumi.network/dev/masumi/core-concepts/transaction-fees
- [S7] Masumi x402 page — https://www.masumi.network/dev/masumi/core-concepts/x402 , https://www.masumi.network/x402 , https://www.masumi.network/vending-machine
- [S8] NMKR × Masumi client page (stats) — https://www.nmkr.io/clients/nmkr-x-masumi
- [S9] Cardano Foundation Masumi case study — https://cardanofoundation.org/case-studies/masumi
- [S10] Catalyst F14 "Masumi AI Agent Network 2.0" — https://projectcatalyst.io/funds/14/cardano-use-cases-partners-and-products/masumi-ai-agent-network-20-by-serviceplan-group
- [S11] Crypto Daily, Cardano joins x402 kit (Sep 2026) — https://cryptodaily.co.uk/2026/09/cardano-joins-x402-kit-ai-agent-payments
- [S12] x402 Cardano exact scheme spec — https://github.com/x402-foundation/x402/blob/main/specs/schemes/exact/scheme_exact_cardano.md ; TS package — https://github.com/x402-foundation/x402/tree/main/typescript/packages/mechanisms/cardano
- [S13] Cardano Foundation x402 facilitator — https://github.com/cardano-foundation/cardano-x402-facilitator
- [S14] Crypto Briefing on x402 SDK (mainnet not yet proven) — https://cryptobriefing.com/cardano-x402-sdk-ada-api-payments/
- [S15] Indigo cardano-mcp — https://glama.ai/mcp/servers/IndigoProtocol/cardano-mcp , https://socket.dev/npm/package/@indigoprotocol/cardano-mcp
- [S16] Mesh AI tools — https://meshjs.dev/ai
- [S17] Cardano Foundation dev skills / AI-assisted development — https://developers.cardano.org/docs/developers/curriculum/start-building/ai-assisted-development/ , https://www.claudepluginhub.com/plugins/cardano-foundation-cardano-dev-skills
- [S18] Intersect DevEx Session 18 resources — https://devex.intersectmbo.org/docs/working-group/q2-2026/sessions/18-cardano-ai-dev-workflow/session-resources
- [S19] Intersect "Production-grade MCP server for Cardano" sessions — https://devex.intersectmbo.org/docs/working-group/q2-2026/sessions/18-cardano-mcp-server/session-notes ; Cardano Unified MCP — https://glama.ai/mcp/servers/easy1staking-com/cardano-unified-mcp-server
- [S20] Midnight "State of the Network – March 2026" — https://midnight.network/blog/state-of-the-network-march-2026
- [S21] Midnight "Midnight network is live" (2026-03-29) — https://midnight.network/blog/midnight-network-is-live ; node operators — https://midnight.network/blog/introducing-midnight-mainnet-trusted-node-operators
- [S22] CaptainAltcoin, permissionless contracts — https://captainaltcoin.com/cardano-news-midnight-enters-its-next-phase-as-smart-contracts-go-permissionless/
- [S23] GN Crypto, NIGHT +27% after mainnet change — https://www.gncrypto.news/news/midnight-night-token-rises-27-percent-after-mainnet-change/ ; The Crypto Gem — https://thecryptogem.substack.com/p/any-developer-can-now-build-on-midnight
- [S24] Midnight docs, Run a validator (FNOs now, SPOs later) — https://docs.midnight.network/validate/run-a-validator
- [S25] Midnight roadmap phases (Kūkolu, Mōhalu, Hua) — https://midnight.network/blog/state-of-the-network-january-2026 , https://midnight.network/blog/guide-to-the-night-token-launch-and-redemption
- [S26] Midnight "State of the Network – August 2026" — https://midnight.network/blog/state-of-the-network-august-2026
- [S27] create-mn-app (npm v0.5.1, 2026-09-01) — https://www.npmjs.com/package/create-mn-app , https://github.com/midnightntwrk/create-mn-app
- [S28] Partner Chains toolkit (archived 2026-04-23) — https://github.com/input-output-hk/partner-chains
- [S29] Charli3 partner chain docs — https://docs.charli3.io/partner-chains
- [S30] Learn Cardano Podcast, "3 partner chains actually building" (2026-04-13) — https://www.castfox.net/podcast/learn-cardano-podcast-3673541/episode/cardanos-ecosystem-just-got-bigger-3-partner-chains-are-actually-building-312123
- [S31] developers.cardano.org AI agents curriculum — https://developers.cardano.org/docs/developers/curriculum/dapps/ai-agents/overview , https://developers.cardano.org/docs/developers/curriculum/dapps/ai-agents/masumi/
- [S32] Kodosumi — https://github.com/masumi-network/kodosumi , https://docs.kodosumi.io
- [S33] Midnight Kapa MCP — https://docs.midnight.network/ai-integration/kapa-mcp-server ; Midnight Expert — https://docs.midnight.network/ai-integration/midnight-expert
- [S34] Sokosumi MCP docs — https://www.masumi.network/dev/sokosumi/mcp ; Sokosumi launch — https://www.house-of-communication.com/de/en/newsroom/2025/06/plan-net-launch-sokosumi.html
- [S35] CriptoNoticias, x402 on Cardano demo (Oct 2025) — https://www.criptonoticias.com/tecnologia/protocolo-x402-coinbase-google-red-cardano/
- [S36] Cardano Forum digest (Apr 28, 2026): "Cardano becomes official x402 chain" — https://forum.cardano.org/t/digest-april-28-2026-cardano-becomes-official-x402-chain-blockfrost-integrates-filecoin-premium-storage-intersect-mbo-governance-technical-updates-cardano-at-jcon-2026-in-cologne-spotlight-on-cardano-content-creators/154318 ; cardano.org digest — https://cardano.org/news/2026-04-28-community-digest/
- [S37] U.Today, Cardano Foundation CEO on AI accountability — https://u.today/cardano-foundation-ceo-calls-attention-to-ai-accountability-gap-whats-missing
- [S38] CoinCentral, Hoskinson defends AI push / Midnight City — https://coincentral.com/cardanos-charles-hoskinson-defends-ai-push-as-midnight-city-expands/
- [S39] HackerNoon, Midnight City — https://hackernoon.com/how-midnight-built-a-living-city-on-blockchain-to-show-you-what-privacy-actually-looks-like
- [S40] Catalyst F14 Aegis proposal — https://forum.cardano.org/t/catalyst-project-aegis-proposal/149875
- [S41] Midnight tokenomics — https://midnight.network/blog/the-tokenomics-powering-midnight-network ; https://beincrypto.com/learn/midnight-tokenomics-explained/
- [S42] Midnight, Guide to NIGHT launch and redemption — https://midnight.network/blog/guide-to-the-night-token-launch-and-redemption
- [S43] Glacier Drop claim portal — https://midnight.network/blog/glacier-drop-claim-portal-now-open
- [S44] Scavenger Mine results (news) — https://cryptonews.net/en/news/altcoins/32090647/
- [S45] Midnight Consensus HK 2026 recap — https://midnight.network/blog/consensus-hk-2026-recap
- [S46] Midnight docs, Deploy and operate / Networks — https://docs.midnight.network/guides/deploy-and-operate , https://docs.midnight.network/guides/networks-and-environments
- [S47] CoinMarketCap Midnight updates (Node 1.0.3, permissionless) — https://coinmarketcap.com/cmc-ai/midnight-network/latest-updates/ ; https://coinmarketcap.com/top-stories/6abfba2e49426d68777d385e/
- [S48] Midnight environments and endpoints — https://docs.midnight.network/relnotes/network
- [S49] Mōhalu / SPO onboarding — https://docs.midnight.network/blog/tags/spos ; https://dev.to/tosh2308/getting-night-tokens-on-midnight-mainnet-a-field-guide-for-the-rest-of-us-4pf0
- [S50] DUST architecture — https://docs.midnight.network/blog/dust-architecture ; Midnight tokens overview — https://docs.midnight.network/tokens/overview
- [S51] DEV tutorial, cNIGHT→mNIGHT MIP-20 bridge — https://dev.to/sammajayi/tutorial-getting-night-tokens-exchanges-bridging-wallet-funding-on-mainnet-21d4
- [S52] Dune Midnight data catalog — https://docs.dune.com/data-catalog/midnight/overview
- [S53] Midnight installation — https://docs.midnight.network/getting-started/installation
- [S54] Midnight support matrix — https://docs.midnight.network/relnotes/support-matrix ; npm `@midnight-ntwrk/compact-runtime`, `@midnight-ntwrk/midnight-js-contracts`
- [S55] Midnight acquire tokens (faucets) — https://docs.midnight.network/guides/acquire-tokens
- [S56] npm `midnight-mcp` (deprecated, migrate to Kapa + Midnight Expert) — https://www.npmjs.com/package/midnight-mcp
- [S57] Midnight Academy (HackQuest) — https://thedefiant.io/news/press-releases/midnight-and-hackquest-launch-midnight-academy-for-global-developers ; Night Sky Accelerator — https://midnight.network/blog/night-sky-accelerator ; MLH — https://events.mlh.com/events/14061-midnight-hackathon-may-2026 ; https://midnight.network/hackathon
- [S58] Midnight "State of the Network – July 2026" — https://midnight.network/blog/state-of-the-network-july-2026
- [S59] example-bboard contract — https://github.com/midnightntwrk/example-bboard (contract/src/bboard.compact)
- [S60] Midnight hello world — https://docs.midnight.network/getting-started/hello-world
- [S61] Midnight "State of the Network – January 2026" — https://midnight.network/blog/state-of-the-network-january-2026
- [S62] IOG partner chains announcements — https://www.iog.io/news/announcing-the-alpha-v1-release-of-the-partner-chains-toolkit , https://www.iog.io/news/partner-chains-are-coming-to-cardano
- [S63] Midnight node release notes (Partner Chains 1.8 in node 0.18) — https://docs.midnight.network/relnotes/node/node-0-18-0
- [S64] Apex Fusion VECTOR — https://crypto.news/apex-fusion-launches-vector-cardanos-institutional-expansion-chain/
- [npm] Package metadata checked 2026-10-07 via registry.npmjs.org: `@x402/cardano` 2.28.0 (2026-09-29), `@indigoprotocol/cardano-mcp` 1.0.11, `create-mn-app` 0.5.1, `midnight-mcp` 0.3.0 (deprecated)
