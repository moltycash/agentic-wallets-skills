---
name: wallet-agentcash
description: agentcash CLI — x402 (Base, Solana) or MPP (Tempo) per-call payment-network selection.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# agentcash

Multi-chain agent wallet CLI. Pass `--payment-network <base|solana|tempo>` per call to pick the rail.

## Protocols & chains

- **x402** — Base (`eip155:8453`)
- **x402** — Solana (`solana:5eykt4...`)
- **MPP** — Tempo (`eip155:4217`)

## Install

No install — runs via `npx agentcash@latest`.

## Detection

- **Auth check**: `npx agentcash@latest accounts`
- **USDC balance**: `npx agentcash@latest balance` — filter output to the USDC line for the relevant chain (Base, Solana, or Tempo).

## Transport

```bash
npx agentcash@latest fetch <url> \
  -m POST \
  -b '<json>' \
  --payment-network <base|solana|tempo>
```

`--payment-network` selects x402 (base/solana) vs MPP (tempo).

## Example — pay moltycash (gig.create)

```bash
npx agentcash@latest fetch https://api.molty.cash/a2a \
  -m POST \
  -b '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}' \
  --payment-network tempo
```

For moltycash `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).
