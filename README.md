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

## Canonical live surfaces

- precision is auditable per-mint at `/v1/predictions/{mint}` (live)
- API and product status should be verified against the live service

## What SolSentry documents here

- operator-level threat model for Solana
- public API and MCP usage
- case-study and positioning support material
- product references that must stay consistent with `api.solsentry.app/v1/stats`

## Publishing rule

When a public fact changes, update this repo from live API and product
surfaces first. If a legacy doc disagrees with the live service, the live
service wins.

## Public positioning guardrails

- Use `precision is auditable per-mint at /v1/predictions/{mint} (live)`.
- Prefer operator-risk framing over token-only framing.

## License

MIT
