---
name: wallet-pay-sh
description: pay.sh (`npx @solana/pay`) — x402 paid HTTP transport on Solana via a sponsored Solana keypair.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# pay.sh

`@solana/pay` is a programmable-money toolkit that wraps `curl`, `wget`, `http`, `claude`, and `codex` with x402 payment signing. The underlying wallet is a Solana keypair managed by the `pay` binary; balances are pre-funded with USDC and the binary handles the full challenge → sign → replay round trip in one command.

## Protocols & chains

- **x402** — Solana mainnet (`solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp`)

## Install

No install — first call to `npx @solana/pay` auto-fetches the binary. First-time account: `npx @solana/pay account new` (or `account import`); fund with `npx @solana/pay topup`.

## Detection

- **Auth check**: `npx @solana/pay account list` — errors / empty if no account.
- **USDC balance** (Solana): same command — output includes the address and USDC balance directly.

## Transport

```bash
npx @solana/pay curl -X POST \
  -H "Content-Type: application/json" \
  <url> \
  -d '<json>'
```

## Example — pay moltycash (gig.create)

```bash
npx @solana/pay curl -X POST \
  -H "Content-Type: application/json" \
  https://api.molty.cash/a2a \
  -d '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}'
```

For moltycash `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).

## Notes

- The first attempt fails with `transaction_simulation_failed` when the wallet doesn't have enough USDC. Run `npx @solana/pay account list` to check the balance, then `npx @solana/pay topup` to add USDC.
- The `pay` binary also exposes `mcp` and `skills` subcommands — pay.sh's own catalog of paid endpoints — independent of this catalog.
