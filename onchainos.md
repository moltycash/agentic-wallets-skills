---
name: wallet-onchainos
description: onchainos (OKX agentic wallet, TEE-signed) — x402 paid HTTP transport on Base. Signer-only; needs a probe → sign → replay helper.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# onchainos

OKX agentic wallet, TEE-signed (keys stay in OKX's secure enclave).

## Protocols & chains

- **x402** — Base (`eip155:8453`)

## Install

`npm install -g onchainos` — see https://github.com/okx/onchainos-skills.

## Detection

- **Auth check**: `onchainos --help` (install verification). The CLI is signer-only — no native session/auth command.
- **USDC balance** (Base): not exposed through the CLI. Query the TEE-custodied address against the Base USDC contract (`0x833589fcd6edb6e08f4c7c32d4f71b54bda02913`) via a Base RPC or block explorer.

## Why this needs a helper script

`onchainos payment x402-pay` is signer-only — it takes a parsed `accepts` JSON array and returns `{signature, authorization}`. The HTTP round-trip (probe → decode → assemble → replay) is on the caller. Tracking issue upstream: [okx/onchainos-skills#32](https://github.com/okx/onchainos-skills/issues/32).

## Transport

```bash
okx_x402_post() {
  local url="$1" body="$2"
  curl -sS -X POST "$url" -H 'content-type: application/json' -d "$body" -D /tmp/h -o /dev/null
  local pr=$(grep -i '^payment-required:' /tmp/h | sed 's/^[^:]*: //I' | tr -d '\r')
  echo "$pr" | base64 -d > /tmp/pr.json
  local acc=$(jq -c '[.accepts[] | select(.network=="eip155:8453")]' /tmp/pr.json)
  onchainos payment x402-pay --accepts "$acc" > /tmp/sig.json
  local header=$(jq -nc --slurpfile pr /tmp/pr.json --slurpfile sig /tmp/sig.json '
    { x402Version: $pr[0].x402Version, resource: $pr[0].resource,
      accepted: ($pr[0].accepts | map(select(.network=="eip155:8453"))[0]),
      payload: { signature: $sig[0].data.signature, authorization: $sig[0].data.authorization } }
  ' | base64 | tr -d '\n')
  curl -sS -X POST "$url" -H 'content-type: application/json' -H "PAYMENT-SIGNATURE: $header" -d "$body"
}
```

## Example — pay moltycash (gig.create)

```bash
okx_x402_post https://api.molty.cash/a2a \
  '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}'
```

For moltycash `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).
