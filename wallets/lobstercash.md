---
name: wallet-lobstercash
description: lobstercash CLI — x402 paid HTTP transport on Base.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# lobstercash

Lobstercash crypto x402 fetch CLI. Signs x402 paid HTTP requests with USDC on Base.

## Protocols & chains

- **x402** — Base (`eip155:8453`)

## Install

`npm install -g lobstercash` — see https://lobster.cash.

## Detection

- **Auth check**: `lobstercash status` — also reports USDC balance per authorized wallet.
- **USDC balance** (Base): same command. Parse the `Balance:` line under each authorized wallet:
  ```bash
  lobstercash status | awk '/Balance:/{ for(i=1;i<=NF;i++) if ($i=="usdc") print $(i-1) }'
  ```
  Output is just the USDC amount(s).

## Transport

```bash
lobstercash crypto x402 fetch <url> --json '<json>'
```

## Example — pay moltycash (gig.create)

```bash
lobstercash crypto x402 fetch https://api.molty.cash/a2a \
  --json '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}'
```

For moltycash `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).
