# solsentry-docs

> RugCheck tells you a fire is burning. SolSentry tells you who lit it.

Public documentation surface for SolSentry: methodology, install flows, API
references, and supporting product material for the operator-intelligence
platform.

Live references:

- Docs surface: `https://solsentry.app/docs`
- API stats: `https://api.solsentry.app/v1/stats`
- GitHub org: `https://github.com/solsentry`
- API infra: `Contabo US-East (66.94.105.185)`, migrated from Hetzner on
  `2026-06-09`
- Frontend/docs hosting: `Cloudflare/Vercel` (not on the VPS)

## Canonical live snapshot

- `83,408` predictions tracked
- `91.4%` aggregate accuracy
- `97.8% CRITICAL precision - auditable per-mint`
- `95.7% HIGH precision`
- `94.6% MEDIUM precision`
- `77.9%` dev wallet coverage
- `~1,492h` continuous runtime
- backend version: `v2.3.21`

> Mint-level metrics above are the canonical, audit-anchored claims (verifiable
> per-mint at `/v1/predictions/{mint}`). Operator-level aggregate counts
> (operators profiled, serial ruggers) are withheld from published headline
> numbers pending a corrected-attribution re-scan (M1).

## What SolSentry documents here

- operator-level threat model for Solana
- public API and MCP usage
- case-study and positioning support material
- product references that must stay consistent with `api.solsentry.app/v1/stats`

## Publishing rule

When a public fact changes, update this repo from the canonical live block
first. If a legacy doc disagrees with `/v1/stats`, `/v1/stats` wins.

## Public positioning guardrails

- Cite the live `critical_precision_pct` from `/v1/stats` (`97.8%` today),
  always framed as `auditable per-mint` — re-verify on the day.
- Prefer operator-risk framing over token-only framing.
- Never publish operator-level aggregate counts as proof until the M1 re-scan.

## License

MIT
