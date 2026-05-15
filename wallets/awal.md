---
name: wallet-awal
description: awal CLI — x402 paid HTTP transport on Base or Solana.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# awal

Multi-chain agent wallet CLI. Signs x402 paid HTTP requests with USDC on Base or Solana.

## Protocols & chains

- **x402** — Base (`eip155:8453`)
- **x402** — Solana (`solana:5eykt4...`)

## Install

No install — runs via `npx awal@latest`.

## Detection

- **Auth check**: `awal status`
- **USDC balance**: `awal balance` — filter output to the USDC line for the relevant chain (Base or Solana).

## Transport

```bash
npx awal@latest x402 pay <url> -X POST -d '<json>' --json
```

## Example — pay moltycash (gig.create)

```bash
npx awal@latest x402 pay https://api.molty.cash/a2a -X POST \
  -d '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}' --json
```

For moltycash `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).
