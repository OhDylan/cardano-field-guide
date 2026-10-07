# Cardano Field Guide — build plan (v1, info-only, no wallet)

Status 2026-10-08: v1 complete, 13 pages built and checked at 375px and 1280px, light and dark.
Research lives in `../research/01–05*.md`.

## Decisions (from user)
- All-English site. Honest tone: show bad data, but always explain it and say what builders can do.
- v1 = information site only. No wallet connect yet. Local version first.
- Name is a placeholder: "Cardano Field Guide".

## Verified TVL story (DefiLlama raw data + daily ADA price)
- USD peak $721M (2024-12-07) was a price spike (ADA ≈ $1.08). "−87% from peak" is true in USD but misleading.
- In ADA terms, TVL was roughly flat at 450–650M ADA from mid-2023 to Jul 2026, and rose in H1 2026 (462M → 560M ADA).
- Real decline: ~512M ADA (Jul 1 2026) → ~270M ADA (Oct 6 2026), about −47% in 3 months, across Minswap, Liqwid and SundaeSwap.
  This coincides with the SecondFi exploit (Jul 8), DeltaDeFi's pause (Jul 15) and an industry-wide DeFi contraction. Treat these as coinciding events, not a proven cause.
- The Sep→Oct USD rebound ($58M → $73M) is mostly price; in ADA it is flat.
- Chart: small multiples (USD TVL / ADA price / TVL in ADA), one axis each, never a dual axis.

## Pages
- `/` home: hero, live stat strip (Koios + DefiLlama with fallback), persona router, unique edges, site map
- `/state`: honest numbers + TVL chart + "what it means for you"
- `/mental-model`: eUTxO vs account, anatomy of a spend, concurrency patterns, Solidity/Anchor Rosetta tables, account-vs-UTxO race simulator (vanilla JS)
- `/stack`: blessed path (Aiken + Mesh/Evolution + Blockfrost/Koios + Yaci DevKit), languages, SDKs, infra, networks/faucets, first-contract walkthrough, AI tooling, pain points
- `/frontiers/{defi,ai-agents,midnight,governance,scaling}`: each split into Live / Coming / Gaps / Try it
- `/funding`: opportunity board (JSON-driven, filterable), decision wizard, PRIME, Catalyst + FC Barcelona case, treasury withdrawal how-to
- `/gaps`: build list (JSON), each gap linked to an RFP and to SDKs that can be composed
- `/ecosystem`: orgs map + node diversity

## Components to build
Badge (live/coming/gap/unverified/open/closed/rolling), Callout, Sources, Stat, PageHeader, NextPrev.

## Facts to re-check before publishing
- Midnight permissionless deploy (news only)
- LayerZero mainnet live
- PRIME RFP process and deadline
- OpenZeppelin proposal outcome (failing at epoch 660)
- Operating Group member: the two research notes spell it CoinCeylon and CoinseLion
