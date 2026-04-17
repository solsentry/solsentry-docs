# SolSentry 🛡️
### Solana's Threat Intelligence Graph — Autonomous Operator-Level Detection

> **"RugCheck tells you a fire is burning. SolSentry tells you who lit it — and where they're going next."**

SolSentry is an autonomous threat intelligence system for Solana that tracks **operators**, not just tokens.  
While existing tools analyze each token or wallet in isolation, SolSentry maps the people behind the scams — across deployments, across wallets, over time.

🔗 **Telegram:** [t.me/solsentryai](https://t.me/solsentryai)  
🔗 **X/Twitter:** [@solsentryai](https://x.com/solsentryai)  
🔗 **Hackathon:** Colosseum Frontier (April–May 2026)

---

## The Problem

Serial scam operators on Solana deploy dozens of rug pulls using different wallets, rotating bot clusters, and paid KOL networks. Each new token looks clean on launch.

Existing tools analyze tokens in isolation — RugCheck flags insider holders within a single token. SolScanner maps wallet connections when you ask. ChainAware scores individual wallet fraud probability. But nobody connects a serial deployer's latest token to their previous attacks.

The gap isn't token-level detection — it's **cross-attack operator intelligence**.

---

## How It Works

```mermaid
flowchart TD
    A[🔍 New Token Detected] --> B
    B["STAGE 1 — Fast Scan
    getAccountInfo + metadata (~2s)
    Authorities · Holders · Concentration
    Known token early-return"]
    B -->|Elevated risk?| C
    C["STAGE 2 — Deep Analysis
    HolderEngine · InsightX · DexScreener
    Serial deployer check
    Social Graph cross-reference"]
    C -->|Confirmed threat?| D
    D["STAGE 3 — Forensic
    Bundle detection (Helius Enhanced TX)
    Drain tracking (10-hop fund flow)
    AI Explainer (multilingual)"]
    D --> E[🚨 ALERT — Telegram Bot]

    style A fill:#1e293b,color:#94a3b8
    style B fill:#0f172a,color:#38bdf8
    style C fill:#0f172a,color:#38bdf8
    style D fill:#0f172a,color:#38bdf8
    style E fill:#7f1d1d,color:#fca5a5
```

---

## Case Study — Operator 4kxscute

SolSentry's hunter agents were already tracking deployer wallet `4kxscute` as part of a **coordinated bot cluster** — wallets sharing SOL funding sources and executing same-block buy patterns across multiple tokens.

When 4kxscute deployed a new token, the hunter fired instantly:

| Detail | Value |
|---|---|
| Risk Score | **100/100** |
| Holders | 1 |
| Top Holder Ownership | 100% |
| Flags | MINT_AUTHORITY_ENABLED, FREEZE_AUTHORITY_ENABLED, TOP_HOLDER_OWNS_100%, VERY_FEW_HOLDERS |
| Detection method | Hunter agent already tracking operator + auto-scan on new deploy |

**No other tool connected this deploy to the operator's previous activity.**  
SolSentry already knew who he was.

### Live Alert Output

![4kxscute Hunter Alert](4kxscute-alert.png)

> Real output from SolSentry's Telegram alert system. Hunter_1570 tracking  
> operator `4kxscute` auto-triggered a scan on the new deployment —  
> returning Risk 100/100 with all critical flags active.

---

## Current Metrics (v2.3.18 — April 17, 2026)

| Metric | Value |
|---|---|
| Token scans executed | **13,131+** |
| Prediction accuracy | **75.9%** |
| False positives at HIGH+ risk | **0** |
| Serial deployers identified | **47** |
| Operators mapped | **778** |
| Bot clusters identified | **1,537** |
| Wallets tracked across operator networks | **8,402** |
| KOL accounts tracked | **99** |
| Hunters active | **30** |
| Continuous runtime | **193h+ (local, no VPS yet)** |
| RPC endpoints | **9 (Helius ×5, Alchemy ×3, Triton ×1)** |
| Tests passing | **318** |
| ALife feedback loops | ✅ Consciousness + MetaLearning wired |

> **On accuracy:** 75.9% reflects **zero confirmed false positives** — every incorrect prediction is a false negative (stealth rugs that launched with clean on-chain metrics and evaded early detection). The system never cries wolf.

---

## ALife Agent Ecosystem

SolSentry uses **Artificial Life principles** — inspired by Tierra, Avida, and Conway's Game of Life — to evolve its detection agents autonomously.

- **30 agents** with 7-gene DNA (spawn_threshold, max_hunters, risk_weight, etc.)
- **Fitness tracking** — agents that predict accurately gain energy and reproduce
- **Genetic mutation** — 30+ real mutations recorded with autonomous timestamps
- **Natural selection** — ineffective agents are culled at 2x population cap
- **Energy metabolism** — -0.3 per tick, births cost 15 energy
- **MetaLearning** — market regime detection (CALM/SURGE/CRASH/MEME) every 120s

This isn't marketing: `genome.json` contains recorded mutations showing parameter changes like `spawn_threshold` evolving from 70 → 95 and `max_hunters` from 10 → 30.

---

## Social Graph of Scam

Three entity types linked across every scan:

| Entity | Description | Count |
|---|---|---|
| **OperatorProfiles** | Serial deployer identities tracked across wallets | 778 operators · 8,402 wallets |
| **BotClusters** | Coordinated wallet groups (same-block buys, shared funding) | 1,537 clusters |
| **ShillNetworks** | KOL-to-operator connections via early-buy patterns | 99 KOLs tracked |

Every scan cross-references the graph. The more the system scans, the harder it is for operators to hide behind new wallets.

---

## Competitive Landscape

| Capability | RugCheck | SolScanner | ChainAware | Blockaid | **SolSentry** |
|---|:---:|:---:|:---:|:---:|:---:|
| Token-level analysis | ✅ | — | — | ✅ | ✅ |
| Wallet graph visualization | ❌ | ✅ | ❌ | ❌ | 🔜 |
| Per-wallet fraud scoring | ❌ | ❌ | ✅ | ❌ | ✅ |
| Transaction simulation | ❌ | ❌ | ❌ | ✅ | ❌ |
| Operator tracking (cross-attack) | ❌ | ❌ | ❌ | ❌ | ✅ |
| Serial deployer detection | ❌ | ❌ | ❌ | ❌ | ✅ |
| Social graph mapping | ❌ | ❌ | ❌ | ❌ | ✅ |
| Bot cluster fingerprinting | ❌ | ✅ | ❌ | ❌ | ✅ |
| KOL-operator correlation | ❌ | ❌ | ❌ | ❌ | ✅ |
| Autonomous detection (24/7) | ❌ | ❌ | ❌ | ✅ | ✅ |
| ALife agent evolution | ❌ | ❌ | ❌ | ❌ | ✅ |
| Drain flow tracking | ❌ | ✅ | ❌ | ❌ | ✅ |

> **SolScanner** shows you the graph when you ask. **SolSentry** builds the graph while you sleep.

---

## Technical Stack

- **Language:** Python 3 (full async architecture)
- **RPC:** Helius (5 keys) · Alchemy (3 keys) · Triton (1 key + WSS)
- **Data Sources:** Helius DAS + Enhanced TX · DexScreener · InsightX · Nansen · Jupiter
- **AI:** Claude Sonnet (multilingual risk explainer, 10 calls/hr)
- **Delivery:** Telegram Bot API (forum topics, real-time alerts)
- **Testing:** 318 tests · 11 test files · 122 commits
- **ALife:** Consciousness wired · MetaLearning auto_adjust · DNA snapshots · hunters_archive
- **Applied:** DD.xyz/Webacy API grant · Helius Startup Launchpad

---

## Roadmap

**Q2 2026** — VPS deployment (24/7) · WebSocket activation (built, pending VPS) · Free public API  
**Q3 2026** — React dashboard v1 (drain map, operator timeline) · x402 micropayments (live v0) · B2B API  
**Q4 2026** — Wallet Reputation Score API · Copy Trade Safety Filter · Launchpad Vetting API  
**2027+** — Cross-chain operator tracking (ETH, BSC) · Institutional compliance API · Seed round

---

## Built By

**Crash Diniz** — Solo founder and developer.  
Self-taught since the early 2000s: Slackware, Unix, Oracle networking. No university, no bootcamp.  
Started learning Python last year — 318 tests, full async architecture, 13,131+ mainnet scans.

> *"Started learning Python last year" is the setup. The metrics above are the punchline.*

**Looking for:** Frontend dev (React dashboard) · Security researcher · BD/growth

---

*Built for the Colosseum Frontier Hackathon · April–May 2026*  
*Powered by Helius · Alchemy · Triton · Claude AI*  
*"Nunca estagnar. Sempre evoluir."*
