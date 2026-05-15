---
name: wallet-tempo
description: tempo CLI request extension — MPP paid HTTP transport on Tempo.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# tempo

Tempo's standalone HTTP+MPP client (the `request` extension on the Tempo CLI). Tempo-only.

## Protocols & chains

- **MPP** — Tempo (`eip155:4217`)

## Install

Tempo CLI from https://docs.tempo.xyz/cli; then `tempo add request` to add the HTTP+MPP extension.

## Detection

- **Auth check**: `tempo wallet whoami`
- **USDC balance** (Tempo): `tempo wallet whoami` — output includes token balances; filter to the USDC line.

## Transport

```bash
tempo request -X POST --json '<json>' <url>
```

## Example — pay moltycash (gig.create)

```bash
tempo request -X POST \
  --json '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}' \
  https://api.molty.cash/a2a
```

For moltycash `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).
