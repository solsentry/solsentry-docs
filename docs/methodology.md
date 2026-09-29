# SolSentry — Detection Methodology

> How SolSentry identifies serial rug operators and scores token risk.
> Last updated: 2026-05-14.

---

## TL;DR

SolSentry watches **operators**, not tokens. Each new Solana token launch is scored across four dimensions — on-chain metadata, holder distribution, bundle patterns, and operator history. The operator history dimension is what existing token-centric tools (RugCheck, GoPlus, Blockaid) don't have: a persistent profile of *the wallet behind the deployment*, built from every prior token that wallet has shipped.

A token deployed by a wallet with 940 prior rugs starts at `risk=100/100` before any on-chain signal is analyzed.

---

## The pipeline — 3 stages, 4 dimensions

```
  new mint ─► stage1_fast (1-2s) ─► stage2_deep (background) ─► stage3_forensic (on-demand)
                   │                        │                           │
                   ▼                        ▼                           ▼
           metadata + dev wallet   holders + liquidity + bundles   10-hop drain trace
```

### Stage 1 — fast screen (~1-2s, always)

| Check | Source | Signal |
|---|---|---|
| Mint authority + freeze authority | Solana RPC `getAccountInfo` (free-tier endpoints first, Helius as fallback) | `MINT_AUTHORITY` flag if mint authority still live post-launch (rug vector) |
| Metadata resolution | Helius DAS / DexScreener | symbol, name, decimals — flag `UNK` for 90s retry |
| Known-token skip | Hardcoded allowlist (SOL, USDC, USD1, DRIFT, etc.) | `risk=10`, no scan |
| Deployer wallet | On-chain token creator | feeds dimension 4 (operator history) |

If stage 1 surfaces a mint/freeze authority flag combined with an operator with ≥2 prior rugs, the pipeline can emit a HIGH-risk alert with **sub-50ms cached operator lookup** + the 6h fast-track resolver window — before stage 2 runs.

### Stage 2 — deep scan (background, 10-30s)

| Dimension | Source | What we extract |
|---|---|---|
| Holders | Helius DAS (on demand) | top-10 concentration, wallet age distribution, insider overlap |
| Liquidity | DexScreener | pool TVL, 24h volume, price change, pool age |
| Bundle patterns | Helius Enhanced TX + getTransactionsForAddress | same-block buyers, tight/wide timing clusters, shared-funder check (payer base-rate, CEX-aware), launch-window buyers when the opening-window observer is enabled |
| Operator history | `operator_profiles.json` | tokens_created, confirmed_rugs, tags, primary_wallet linkage |

### Stage 3 — forensic (manual, heavy)

Used post-rug or on-demand via `/v1/drain-trace/{wallet}`:
- Follow SOL flow up to 10 hops
- Classify destination wallets (CEX, mixer, bridge, burner)
- Tag hops for recovery feasibility

---

## The 4 risk dimensions (composition)

```
  risk_score = f(token_integrity, holder_distribution, bundle_signature, operator_history)
```

Each dimension scores 0-100. Final risk is a weighted blend, subject to a **serial-deployer boost** (+15 or +25 if `operator.total_rugs >= 2` or `>= 5` respectively).

### Dimension 1 — Token integrity
Mint authority live post-launch, freeze authority, metadata anomalies, zero-decimal tokens, honeypot-style transfer restrictions.

### Dimension 2 — Holder distribution
Top-10 holder concentration, wallet age of top holders (freshly-funded wallets = red flag), overlap with known sniper/insider clusters.

### Dimension 3 — Bundle signature
Same-block buy/sell patterns, common funding source (shared first-funder, weighted by the payer's base-rate so exchange hot wallets don't count as coordination), and repeat-cluster wallets. Cross-referenced against a running registry of wallets seen in coordinated clusters across tokens; wallets whose breadth shows they buy every launch are discounted, not counted as bundlers.

### Dimension 4 — Operator history (the differentiator)
Every deployer wallet has an `OperatorProfile`:
- `primary_wallet` + `alt_wallets` (linked via funding lineage)
- `tokens_created` (full mint list)
- `confirmed_rugs` count + rug rate
- `tags` (fast_deployer, rebrand_artist, pump_fun_farmer, etc.)
- Bot cluster membership (frozenset-indexed, O(1) lookup)

A wallet deploying its 941st rug gets this dimension at 100 regardless of what the token looks like.

---

## Cluster detection — how we link wallets across rugs

Naive approach: track each wallet individually. Problem: operators rotate wallets every few days.

SolSentry uses three overlapping signals:
1. **Funding lineage** — same-source SOL funding across multiple deployers (common CEX withdrawal → 5 deployer wallets funded in same hour)
2. **Deployment cadence** — same-minute deployments on the same launchpad pattern (pump.fun graduate → raydium migration timing)
3. **Bundle co-occurrence** — same bundle wallets appearing as first-buyers across deployments

When 3+ wallets correlate on 2+ signals, they become a cluster. Cluster membership is stored as `frozenset` in `intelligence.json::bot_clusters` with O(1) reverse lookup (`wallet → cluster_ids`). Adding a new mint to an existing cluster propagates the aggregate rug rate to every member.

Some clusters contain 60+ deployer wallets with thousands of cross-attributed tokens. Current cluster counts: `/v1/stats`.

---

## Outcome resolution — how accuracy is measured

Every risk score predicted by the pipeline is stored as an `OutcomePrediction` with:
- `predicted_risk` (0-100)
- `predicted_at` (timestamp)
- `dna_snapshot` (agent DNA at prediction time — feeds MetaLearning)
- `final_outcome` (initially `pending`)

After a resolution window (2 days primary, 14 days safe-recheck, immediate for volume-dead tokens), the pipeline re-scans:
- `confirmed_scam` = rug_pulled OR honeypot OR liquidity collapse OR token death + pre-launch risk ≥70
- `confirmed_safe` = alive + price/liquidity stable + no rug signals post-window
- `pending` = insufficient signal yet

Accuracy is then `sum(was_correct=True) / resolved`.

### CRITICAL precision profile

Precision is reported per tier and read live from `/v1/stats` (`precision_by_tier`) — never quoted from this page. The breakdown below is the **May 14, 2026 audit snapshot**, kept for method, not as a current figure: at CRITICAL severity there were 607 FP events across 231 unique mints, composed as:

- **228 threshold edge cases** — tokens surviving 1–14 days post-flag (most resolve as rugs eventually, but cross the 14-day safe-recheck window first)
- **3 unclassified long survivors** — tokens that have survived past the standard windows, still flagged CRITICAL, outcome ambiguous
- **2 high-frequency rescan patterns** — two specific mints (`9NwxDyev` 117× and `BBKPiLM9` 58×) account for 92 FP events between them; this is a scan-dedup tech-debt issue scheduled for v2.4

In that snapshot every CRITICAL FP was a threshold edge case rather than a false-alarm pattern. Precision is high, not perfect; current values and every individual call are auditable per mint.


Full per-mint audit available at `/v1/predictions/{mint}`. Reproducibility methodology in `docs/accuracy_audit.md`.

---

## Live numbers

Every figure (predictions issued, accuracy, per-tier precision, runtime) is served live — fetch it rather than copying it: `curl https://api.solsentry.app/v1/stats`



---

## Limitations (honest framing)

- **Pre-launch blindspot:** before a wallet deploys its 2nd token, SolSentry has no operator history — it falls back to dimensions 1-3 only. First-time rugs are the stealth-rug category where false negatives concentrate.
- **Indexing lag:** fresh tokens can take 30s-10min to appear on DAS/DexScreener. The pipeline retries at 90s/300s/900s for UNK symbols, but a rug executed inside the first minute will escape the deep scan.
- **Operator graph requires funding traceability:** mixers and peel chains (10+ hops through cold wallets) break the funding-lineage signal. The drain-trace endpoint surfaces up to 10 hops but classification beyond that is inferential.
- **Not a trading signal:** SolSentry scores deployment risk, not price trajectory. A risk=10 token can still rug via exploit; a risk=100 token can still moon briefly before rugging.

---

*SolSentry is live on Solana mainnet. API: `api.solsentry.app`. Package: `@solsentry/mcp`. All numbers above are queryable through the API.*
