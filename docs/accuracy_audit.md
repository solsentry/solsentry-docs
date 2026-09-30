# SolSentry — Accuracy Audit: Method

> How a prediction's outcome label is resolved, what enters and leaves the resolved universe, and where to audit individual calls.
> This document describes the **method only**. It intentionally contains no measured figures.
> Companion: [`docs/methodology.md`](./methodology.md) (the detection pipeline).

> **Note.** Precision must be read next to the base rate of the resolved universe; figures are being re-measured.

---

## What is measured

Every risk score the pipeline emits is stored as a prediction with a timestamp, a risk score, a severity band, and an initial outcome of `pending`. After a resolution window the pipeline re-scans the token and assigns a final outcome. Two quantities are derived from resolved predictions:

- **Aggregate accuracy** — the share of resolved predictions labelled correct, across all severity bands.
- **Per-tier precision** — within one severity band (for example CRITICAL), the share of predictions in that band labelled correct.

Neither quantity is quoted on this page. Interpret any figure together with the base rate (see below).

---

## How the outcome label is resolved

### Final outcome

After the resolution window (a primary window, a longer safe-recheck window, and an immediate path for volume-dead tokens) the pipeline assigns one of:

- `confirmed_scam` — rug pulled, honeypot, liquidity collapse, or token death, together with a high pre-launch risk score.
- `confirmed_safe` — token alive, price and liquidity stable, no rug signals after the window.
- `pending` — not enough signal yet.

### Correctness label

`was_correct` is derived from the prediction and the final outcome:

- `predicted_risk ≥ 50` and `final_outcome ∈ {confirmed_scam, honeypot, rug_pulled}` → correct
- `predicted_risk < 50` and `final_outcome = confirmed_safe` → correct
- otherwise → incorrect

Accuracy is `correct / resolved`. Precision at a tier is `correct in tier / resolved in tier`. The denominator counts prediction events, not unique mints: a mint scanned more than once contributes more than once.

The resolver that produces these labels is being reworked, so the label definition above describes the current method and may change.

---

## What enters and what leaves the resolved universe

**Enters:** predictions whose final outcome is `confirmed_scam` or `confirmed_safe`.

**Leaves (excluded from accuracy and precision):**

- `pending` predictions.
- Predictions whose outcome cannot be established from on-chain evidence within the windows.
- Known-token allowlist entries (large, established tokens) that are skipped rather than scanned.

The composition of the resolved universe matters. If most resolved tokens are scams, a scorer that flags nearly everything will show high precision without adding information. Read every precision figure next to the base rate of the resolved universe, and prefer per-mint inspection over aggregates.

---

## Known limits of the label

- **Window effects.** A token that survives a window but rugs later is labelled `confirmed_safe` at resolution time. Longer windows change the label.
- **Stealth rugs.** First-time deployers with clean metadata, distributed holders, and cold-funded wallets score low on dimensions 1–3 and have no operator-history evidence, so they tend to resolve as missed calls rather than false alarms.
- **Repeated scans.** Rescans of the same mint are separate events unless deduplicated.
- **Scope.** Labels measure operator-deployment risk on Solana. They are not comparable to fund-flow classification or contract-level safety benchmarks, which measure different things.

---

## Reproducing counts from the prediction store

```python
import json
with open("store/outcome_predictions.json") as f:
    preds = json.load(f)["predictions"]

# CRITICAL predictions that resolved as confirmed_safe (labelled incorrect)
fp_critical_events = [
    p for p in preds.values()
    if p.get("severity") == "CRITICAL"
    and p.get("final_outcome") == "confirmed_safe"
    and p.get("was_correct") is False
]
unique_fp_mints = {p["mint"] for p in fp_critical_events}

print(f"Events: {len(fp_critical_events)}")
print(f"Unique mints: {len(unique_fp_mints)}")
```

Compare the event count with the unique-mint count, and compare the flagged set with the base rate of scams among all resolved predictions, before drawing any conclusion.

---

## Data consistency

Three persistent stores must stay in sync (enforced by tests and by `/health/invariants`):

1. `outcome_predictions.json` — raw predictions, resolved outcomes, severity bands.
2. `operator_profiles.json` — per-operator aggregates.
3. `intelligence.json::dev_wallets[wallet].rugs` — runtime-indexed rug lists per wallet.

Invariants checked on every call to `/health/invariants`:

- **INV-01:** every `confirmed_scam` prediction has a `dev_wallet` or `creator_address`.
- **INV-02:** every operator's `confirmed_rugs` count matches the rug list in `intelligence.json`.
- **INV-03:** every `confirmed_scam` mint in predictions appears in its dev wallet's rug list.

Divergence is a bug.

---

## Where to audit

| What | How |
|---|---|
| Per-mint outcome and history | `curl https://api.solsentry.app/v1/predictions/{mint}` |
| Invariant health check | `curl https://api.solsentry.app/health/invariants` |
| Operator profile | `curl https://api.solsentry.app/v1/operator/{wallet}` |

Each prediction is auditable per mint. Any figure drifts as predictions resolve, so treat a quoted value as dated, never as a cached truth.
