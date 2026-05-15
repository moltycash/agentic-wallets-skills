---
name: wallet-purl
description: purl CLI — auto-detects x402 (Base, Solana) vs MPP (Tempo) per call.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# purl

Auto-detects whether the paid endpoint uses x402 or MPP and signs accordingly. Single-shot URL invocation.

## Protocols & chains

- **x402** — Base (`eip155:8453`), Solana (`solana:5eykt4...`)
- **MPP** — Tempo (`eip155:4217`)

## Detection

- **Auth check**: `purl wallet list`
- **USDC balance**: `purl balance` — filter output to the USDC line for the relevant chain (Base, Solana, or Tempo).

## Transport

```bash
purl <url> -X POST --json '<json>'
```

## Example — pay moltycash (gig.create)

```bash
purl https://api.molty.cash/a2a -X POST \
  --json '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}'
```

For moltycash `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).
