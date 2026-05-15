---
name: wallet-bankr
description: bankr CLI — x402 paid HTTP transport on Base.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# bankr

Bankr agent CLI. Signs x402 paid HTTP requests with USDC on Base.

## Protocols & chains

- **x402** — Base (`eip155:8453`)

## Install

`npm install -g bankr` — see https://docs.bankr.bot.

## Detection

- **Auth check**: `bankr whoami`
- **USDC balance** (Base):
  ```bash
  bankr wallet portfolio --chain base --json \
    | jq '.balances.base.tokenBalances[]? | select(.token.baseToken.symbol == "USDC") | .token.balance'
  ```
  Empty output = no USDC. Replace `base` with `solana` for SVM-USDC if needed.

## Transport

```bash
bankr x402 call <url> --method POST --max-payment <n> --body '<json>'
```

`--max-payment` must be ≥ payment amount + endpoint fee. bankr defaults to `$1` if omitted; pick a value with headroom.

## Example — pay moltycash (gig.create)

```bash
bankr x402 call https://api.molty.cash/a2a \
  --method POST --max-payment 1.10 \
  --body '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}'
```

For moltycash `tip` / `hire` payloads and fee rules, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).

## Notes

- The `--max-payment` quirk is a bankr-specific safety cap; other x402 CLIs in this catalog don't require it.
