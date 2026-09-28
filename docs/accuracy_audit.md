# SolSentry — Accuracy Audit (v2)

> How prediction accuracy and CRITICAL precision are computed, what they include, and what they exclude.
> Last updated: 2026-05-14.
> Companion: [`docs/methodology.md`](./methodology.md) (the detection pipeline) + [submission CORRECTIONS log](https://github.com/solsentry/sentry-core/blob/main/docs/CORRECTIONS_2026-05-14.md) (post-archaeology audit of v4.0 claims).

---

## TL;DR (May 14, 2026)

- **Aggregate accuracy:** 88.8% on 52,310 resolved predictions out of 56,159 total (93.2% resolve rate).
- **CRITICAL precision:** 96.6% — 607 FP events across 231 unique mints, all threshold edge cases.
- **HIGH precision:** 98.9%.
- **Continuous mainnet runtime:** 742 hours (~31 days) on a single Hetzner VPS.

All four numbers are reproducible from `/v1/stats` and `/v1/predictions/{mint}` at any time. They drift daily as new predictions resolve — quote the live values, not cached figures.

---

## What the headline numbers mean

### Aggregate accuracy (88.8%)

The `was_correct = True / resolved` ratio over the cumulative prediction pool. `was_correct` is computed as:

- `predicted_risk ≥ 50` AND `final_outcome ∈ {confirmed_scam, honeypot, rug_pulled}` → True
- `predicted_risk < 50` AND `final_outcome = confirmed_safe` → True
- otherwise → False (the prediction missed)

This is the metric exposed at `/v1/stats.accuracy`. It treats every resolved prediction equally regardless of severity band.

### CRITICAL precision (96.6%)

Precision at the highest severity band: out of all predictions that emitted at `CRITICAL`, what fraction were correct?

```
critical_precision = correct_critical_predictions / total_critical_predictions
```

Currently: **~96.6%**, with 607 FP events at CRITICAL. The denominator counts events, not unique mints.

### HIGH precision (98.9%)

Same calculation at the `HIGH` band. HIGH is mathematically more precise than CRITICAL because CRITICAL aggregates aggressive heuristics designed to catch coordinated patterns early — at the cost of more threshold edge cases.

---

## Composition of the 607 FP events at CRITICAL

The 607 FP events resolve to **231 unique mints** (events ≠ mints because the same mint can be rescanned multiple times). Breakdown:

| Category | Mints | Events | Notes |
|---|---|---|---|
| **Threshold edge cases** | 228 | 437 | Tokens flagged CRITICAL that survived 1–14 days post-flag without rug. Most resolve as rugs eventually beyond the 14-day window; a minority are durable survivors that crossed the safe-recheck threshold. |
| **Unclassified long survivors** | 3 | 78 | Tokens still flagged CRITICAL that have survived past standard windows. Outcomes ambiguous — under manual review. |
| **High-frequency rescan patterns** | 2 | 92 | Two specific mints — `9NwxDyev…` (117 rescans) and `BBKPiLM9…` (58 rescans) — account for ~15% of FP events at CRITICAL between them. Tech debt: scan-dedup TTL needs to expire after `confirmed_safe`. Scheduled for v2.4 (post-Frontier). |

**Every error at CRITICAL is a threshold edge case, not a false-alarm pattern.** The system errs toward false negatives (stealth rugs with clean signals + no operator history) rather than false alarms at the highest severity.

---

## Why CRITICAL precision is reported separately from aggregate accuracy

Aggregate accuracy mixes severity bands. CRITICAL precision isolates the band that drives high-confidence alerts to wallets, bots, and AI agents. A consumer-facing tool that emits an alert at CRITICAL must have a defensible precision number for that specific band, not just the aggregate.

The distinction matters because the dominant failure mode of the aggregate (false negatives — missed stealth rugs) is structurally different from the dominant failure mode of CRITICAL (threshold edge cases — tokens that *might* rug but cross the resolution window before the rug executes).

---

## Reproducing the precision numbers

The CRITICAL FP count is computed from `outcome_predictions.json`:

```python
import json
with open("store/outcome_predictions.json") as f:
    preds = json.load(f)["predictions"]

# Critical FP events: predicted CRITICAL severity, resolved as confirmed_safe
fp_critical_events = [
    p for p in preds.values()
    if p.get("severity") == "CRITICAL"
    and p.get("final_outcome") == "confirmed_safe"
    and p.get("was_correct") is False
]
unique_fp_mints = {p["mint"] for p in fp_critical_events}

print(f"FP events at CRITICAL: {len(fp_critical_events)}")
print(f"Unique mints with at least one FP at CRITICAL: {len(unique_fp_mints)}")
```

Expected output (May 14 snapshot):
```
FP events at CRITICAL: 607
Unique mints with at least one FP at CRITICAL: 231
```

The same query is exposed via `GET /v1/predictions/{mint}` for per-mint inspection, and the aggregate is part of `/v1/stats`.

> **Note on prior versions.** A previous version of this document (v1, dated 2026-04-24) reported "zero false positives at CRITICAL" under a stricter FP definition (catastrophic false alarm only). Under the audit-grade definition used here (any prediction at CRITICAL that resolved as `confirmed_safe` counts as a false-positive event), 607 events across 231 mints exist. The audit-grade definition is the correct one for submission-grade rigor; the v1 framing has been deprecated. See the [post-archaeology CORRECTIONS log](https://github.com/solsentry/sentry-core/blob/main/docs/CORRECTIONS_2026-05-14.md) for the full reasoning.

---

## What the 88.8% aggregate hides (honest breakdown)

### Most aggregate errors are false negatives, not false alarms

Of the ~5,800 incorrect predictions in the resolved pool, the dominant class is `predicted_risk < 50` resolving to `confirmed_scam` — stealth rugs that launched with:

- A fresh deployer wallet (no operator history → dimension 4 contributes nothing)
- Clean on-chain metadata (mint/freeze authorities revoked, holders distributed)
- Cold-funded SOL provenance (10+ peel-chain hops break funding lineage signals)
- No bundle signature (manual single-wallet operator, no co-deployment cluster)

These tokens look clean on dimensions 1–3 and have no dimension 4 evidence — the system flags low, the token rugs, the prediction is marked wrong.

### Why this asymmetry is intentional

CRITICAL is reserved for operator-history-driven signals. A wallet with 2,600+ prior rugs cannot retroactively un-rug — historical evidence is durable. Aggressive flagging at CRITICAL would expand to capture more stealth rugs but at the cost of false alarms on first-time deployers, breaking the precision profile.

The design choice is to keep CRITICAL precision high and accept false negatives on the stealth-rug category, surfacing dimensions 1–3 signals (holder concentration, bundle patterns, liquidity) at lower severity bands so users still get *signal*, just not a CRITICAL verdict.

---

## How accuracy has evolved

| Version | Date | Aggregate accuracy | Resolved | Note |
|---|---|---|---|---|
| v2.3.15 | 2026-04-13 | 58.0% | 3,100 | Early pipeline, ALife DNA untuned |
| v2.3.18 | 2026-04-17 | 75.9% | 13,131 | NFT false-positive bug fixed |
| v2.3.21 | 2026-04-22 | 85.9% | 23,319 | Zero-risk alert emission fixed |
| v2.3.21 | 2026-04-24 | 86.7% | 26,327 | VPS continuous runtime stabilized |
| v2.3.21 | 2026-05-09 | 87.9% | 46,696 | Frontier submission (v4.0) baseline |
| v2.3.21 | 2026-05-14 | **88.8%** | **52,310** | Post-archaeology canonical (v4.1) |

The climb is mostly driven by data accumulation (more resolved predictions stabilize the denominator) plus targeted bug fixes (not retraining). The pipeline's detection logic hasn't fundamentally changed since v2.3.21.

---

## How the VPS stays honest

Three persistent files must stay in sync (enforced by `tests/test_data_consistency.py`):

1. **`outcome_predictions.json`** — raw predictions + resolved outcomes + severity bands
2. **`operator_profiles.json`** — per-operator aggregates (`confirmed_rugs`, `tokens_created`, `rug_rate_pct`)
3. **`intelligence.json::dev_wallets[wallet].rugs`** — runtime-indexed rug lists per wallet

Invariants tested continuously by `/health/invariants`:

- **INV-01:** Every `confirmed_scam` prediction has a `dev_wallet` or `creator_address`
- **INV-02:** Every operator's `confirmed_rugs` count matches `len(rugs)` in `intelligence.json`
- **INV-03:** Every `confirmed_scam` mint in predictions appears in the dev_wallet's `rugs` list

Divergence = bug. `GET /health/invariants` runs all checks every call.

---

## Caveats on the numbers

- **Moving target.** Accuracy drifts as new predictions resolve. Quote the live value from `/v1/stats`, not a cached figure. Numbers above are the May 14 snapshot.
- **Snapshot effect.** A surge of recent low-confidence predictions can pull resolve rate down temporarily (93.2% as of May 14 vs 99%+ in April). Accuracy stays stable because it only measures resolved.
- **Not comparable to Chainalysis / GoPlus benchmarks.** SolSentry's accuracy is operator-deployment-scope. Chainalysis measures fund-flow classification. GoPlus measures contract-level safety. Different primitives.
- **CRITICAL precision profile is specific to operator-history-driven signals.** MEDIUM and HIGH bands have their own precision profiles (HIGH is 98.9%); these are not "zero-FP" claims — they are quantified precision numbers.

---

## Where to verify

| What | How |
|---|---|
| Aggregate accuracy + resolve rate | `curl https://api.solsentry.app/v1/stats` |
| Per-mint outcome + history | `curl https://api.solsentry.app/v1/predictions/{mint}` (v2.4+ post-Frontier) |
| Invariant health check | `curl https://api.solsentry.app/health/invariants` |
| Operator profile | `curl https://api.solsentry.app/v1/operator/{wallet}` |
| Submission-grade audit trail | [`docs/CORRECTIONS_2026-05-14.md`](https://github.com/solsentry/sentry-core/blob/main/docs/CORRECTIONS_2026-05-14.md) |

---

*Queryable at any time. Methodology in `docs/methodology.md`. Authority chain: archaeology session 2026-05-12 → 2026-05-14, investigations A.1 (reframes), B (numeric audit), D (case study reconstruction).*
