SolSentry 🛡️
Solana Threat Intelligence — Operator-Level Rug Pull Detection
"The 62nd token looked clean. The operator's history didn't."

SolSentry is a Solana threat intelligence system that tracks operators, not just tokens.
While existing tools (RugCheck, GoPlus, DEXScreener) analyze each token in isolation, SolSentry maps the wallets behind them — across deployments, over time.

The Problem
Serial rug pull operators deploy dozens of scam tokens from the same wallet cluster.
Each new token looks clean on launch. The threat signal lives in the operator's history — invisible to every per-token tool on the market.

How It Works
text
New Token Detected
       │
       ▼
┌─────────────────────────────────┐
│  STAGE 1 — GUARDIAN Score       │
│  12 behavioral heuristics       │
│  Liquidity, holders, mint auth, │
│  freeze auth, LP lock, deployer │
└──────────────┬──────────────────┘
               │ Elevated risk?
               ▼
┌─────────────────────────────────┐
│  STAGE 2 — Social Graph         │
│  Maps deployer → drain wallets  │
│  → KOL accounts across tokens   │
│  Serial operator detection      │
└──────────────┬──────────────────┘
               │ Confirmed threat?
               ▼
┌─────────────────────────────────┐
│  STAGE 3 — Continuous Monitor   │
│  WebSocket real-time tracking   │
│  Behavioral shift detection     │
└─────────────────────────────────┘
Case Study — Operator 4kxscute
On March 12, 2026, SolSentry's social graph flagged deployer wallet 4kxscute as a serial operator.

Event	Time
Token #62 deployed by 4kxscute	T+0:00
SolSentry HIGH RISK alert issued	T+0:04
Token appears on aggregator risk radar	T+0:27
Rug pull executed	T+0:23
The token had clean static metadata at launch.
The threat signal was in the operator's history across 61 prior confirmed rug pulls — not in the token itself.
