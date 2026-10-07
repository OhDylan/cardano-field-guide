# Cardano Field Guide

An independent, honest, technical field guide to building on Cardano (October 2026), for engineers coming from Ethereum, Solana, web2 and AI.

- `site/`: the Astro static site (all English). `cd site && npm install && npm run dev`, then open http://localhost:4321
- `research/`: sourced research notes (2026-10-07) and raw DefiLlama data used for the TVL analysis
- `vercel.json`: builds `site/` and serves `site/dist`

Data that people will want to update lives in `site/src/data/` (`opportunities.json`, `gaps.json`, `tvl-history.json`).
