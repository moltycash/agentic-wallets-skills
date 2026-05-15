---
name: wallet-moonpay
description: moonpay CLI — x402 paid HTTP transport on Solana.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# moonpay

Moonpay agent wallet CLI. x402 transport via `moonpay x402 request`.

## Protocols & chains

- **x402** — Solana (`solana:5eykt4...`)

## Install

`npm install -g @moonpay/cli` — see https://developer.moonpay.com. First-time auth: `moonpay login` then `moonpay verify`; wallet via `moonpay wallet create` or `moonpay wallet import`.

## Detection

- **Auth check**: `moonpay user retrieve` — errors with "Token refresh failed" if not logged in. `moonpay wallet list` is a softer install/wallet check that works even with stale tokens.
- **USDC balance** (Solana):
  ```bash
  moonpay --json token balance list --wallet agent-wallet --chain solana \
    | jq '.items[] | select(.symbol == "USDC") | .balance.amount'
  ```
  Empty output = no USDC. Adjust `--chain` for other chains; the `symbol == "USDC"` filter stays the same.

## Transport

```bash
moonpay x402 request \
  --method POST \
  --url "<url>" \
  --body '<json>' \
  --wallet <wallet> --chain solana
```

## Example — pay moltycash (gig.create)

```bash
moonpay x402 request \
  --method POST \
  --url "https://api.molty.cash/a2a" \
  --body '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}' \
  --wallet agent-wallet --chain solana
```

For moltycash `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).
