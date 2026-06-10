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

- `80,017` predictions tracked
- `91.2%` aggregate accuracy
- `97.9% CRITICAL precision - auditable per-mint`
- `95.3% HIGH precision`
- `94.4% MEDIUM precision`
- `10,112` operators profiled
- `7,004` serial ruggers identified
- `78.0%` dev wallet coverage
- `~1,367h` continuous runtime
- backend version: `v2.3.21`

## What SolSentry documents here

- operator-level threat model for Solana
- public API and MCP usage
- case-study and positioning support material
- product references that must stay consistent with `api.solsentry.app/v1/stats`

## Publishing rule

When a public fact changes, update this repo from the canonical live block
first. If a legacy doc disagrees with `/v1/stats`, `/v1/stats` wins.

## Public positioning guardrails

- Use `97.9% CRITICAL precision - auditable per-mint`.
- Prefer operator-risk framing over token-only framing.

## License

MIT
