---
name: wallet-circle
description: Circle CLI — x402 paid HTTP transport on Base via Circle smart-account wallets.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# circle

Circle's agent wallet CLI. Email-OTP auth, server-custodied smart accounts, paymaster-sponsored gas.

## Protocols & chains

- **x402** — Base (`eip155:8453`)

## Install

`npm install -g @circle-fin/cli` — see https://developers.circle.com. First-time auth requires email OTP, agent-wallet creation, and a one-shot transfer to deploy the smart account before x402.

## Detection

- **Auth check**: `circle wallet list --type agent --chain BASE --output json` (errors if not logged in).
- **USDC balance** (Base): no native CLI command — query the address returned by `wallet list` against the Base USDC contract (`0x833589fcd6edb6e08f4c7c32d4f71b54bda02913`) via a Base RPC or block explorer.

## Transport

```bash
circle services pay "<url>" \
  --address <wallet> --chain BASE \
  --data '<json>'
```

## Example — pay moltycash (gig.create)

```bash
circle services pay "https://api.molty.cash/a2a" \
  --address 0xWALLET --chain BASE \
  --data '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}'
```

For moltycash `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).

## Notes

- Smart account must be deployed on-chain (step 6 above) before the first `circle services pay` call.
