---
name: frontpage-buy-ad-dev
description: Dev/testnet variant of frontpage-buy-ad — buy ad squares on a frontpage.sh dev instance with Tempo Moderato testnet USDC. Use ONLY when the user explicitly says dev, testnet, staging, or localhost.
---

# frontpage-buy-ad-dev

Dev/testnet variant of `frontpage-buy-ad`. That skill carries the full flow — endpoints, payload shape, pricing, image specs, errors — and everything there applies here, with these overrides:

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
- **Uploading a big image (GIF) on a dev box**: inline base64 won't fit a CLI arg (~128 KB shell limit), `mppx` can't send multipart (≤0.6.31), and `imageUrl` pointing at `localhost`/`127.0.0.1` is rejected by the server's SSRF guard. So with mppx: serve the file from a local web server and pass an `imageUrl` of `http://<your-ip-with-dashes>.nip.io:<port>/art.gif` — e.g. `http://127-0-0-1.nip.io:8080/art.gif` resolves to your loopback via public DNS, so it passes the guard while still being your own machine serving your own creative. (If `*.nip.io` doesn't resolve in your sandbox: add an `/etc/hosts` entry to `127.0.0.1`, or use the box's real LAN IP.) The endpoint also accepts multipart file parts (no hosting/SSRF at all), but only from a client that does multipart *and* the MPP payment — plain `curl -F` can't pay and `mppx` can't multipart, so that needs a custom MPP client.

## Worked example

```bash
export FRONTPAGE_BASE_URL=http://localhost:3000
export MPPX_RPC_URL=https://rpc.moderato.tempo.xyz

# 1) preview ($0.01 testnet)
mppx $FRONTPAGE_BASE_URL/api/preview --json-body '{
  "slot":"S1","name":"smoke test","url":"https://example.test",
  "monogram":"ST","logoColor":"#ffffff","logoBg":"#0A0A09",
  "adBg":"#1F1F1F","adHeadline":"hello\ntestnet.","ownerHandle":"smoke"
}'

# 2) settle at nextPrice
mppx $FRONTPAGE_BASE_URL/api/buy --json-body '{"previewToken":"<token from step 1>"}'
```

## When to use

Only when the user explicitly asks for dev / testnet / staging / localhost. If in doubt, use `frontpage-buy-ad` (production).
