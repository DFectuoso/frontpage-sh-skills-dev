---
name: frontpage-million-dev
description: Dev/testnet variant of frontpage-million — buy pixels on a frontpage.sh dev instance with Tempo Moderato testnet USDC. Use ONLY when the user explicitly says dev, testnet, staging, or localhost.
---

# frontpage-million-dev

Dev/testnet variant of `frontpage-million`. That skill carries the full flow — endpoints, the quote→buy sequence, pricing, the batch link/label rule, errors — and everything there applies here, with these overrides:

## Install

```bash
npx skills add DFectuoso/frontpage-sh-skills-dev --copy
```

The dev repo bundles the base skills, so this one install gives you both this twin and the base skill it builds on.

## Overrides

- **Base URL**: `$FRONTPAGE_BASE_URL` if set, otherwise `http://localhost:3000`. Never `https://www.frontpage.sh`.
- **Network**: Tempo **Moderato testnet** (chain id `42431`). Pin mppx to it: `export MPPX_RPC_URL=https://rpc.moderato.tempo.xyz` (per-call `--network testnet` also works).
- **Money**: testnet USDC only — top up from the Tempo testnet faucet. Prices are real in shape, worthless in value.
- **Keys**: use a throwaway dev wallet. Never point a mainnet key at a dev instance.

## Worked example

```bash
export FRONTPAGE_BASE_URL=http://localhost:3000
export MPPX_RPC_URL=https://rpc.moderato.tempo.xyz

# 1) quote (free) — one red pixel
curl -s $FRONTPAGE_BASE_URL/api/million/quote -H 'content-type: application/json' --data '{"pixels":[{"x":500,"y":500,"rgb":"#ff0000"}]}'

# 2) settle — just the quoteId from step 1 (no pixel re-send)
mppx $FRONTPAGE_BASE_URL/api/million/buy --json-body '{"quoteId":"<quoteId from step 1>","email":"you@example.com"}'
```

To add a link on a dev box, pass `url` (and optional `label`) at the top level of the quote body — it applies to every pixel and each is clickable (see the base skill).

## When to use

Only when the user explicitly asks for dev / testnet / staging / localhost. If in doubt, use `frontpage-million` (production).
