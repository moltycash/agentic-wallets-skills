---
name: agentic-wallets
description: Detect installed/authenticated wallet CLIs and send x402 / MPP paid HTTP requests via them. Covers 10 wallets across Base, Solana, World Chain, SKALE, Tempo, Stellar, Monad.
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# agentic-wallets — generic x402 / MPP transport skill

A catalog of wallet CLIs that can sign **x402** (HTTP 402) or **MPP** (Multi-Provider Payments) paid HTTP requests, with detection probes and per-chain balance commands.

## Protocols & chains supported by the catalog

- **x402**: Base (`eip155:8453`), Solana (`solana:5eykt4...`)
- **MPP**: Tempo (`eip155:4217`)

## How to use

1. Run the probe below to see which wallet CLIs are installed/authed and what USDC balance exists.
2. Match the chain you need (from the paid endpoint's `accepts[].network`) against the wallet's `Protocols & chains` section.
3. Fetch the chosen wallet's doc for its exact transport command.
4. Wrap your own JSON / form body in the transport pattern.

If none of the catalog wallets are present, the agent decides whether to install one. Each per-wallet doc carries a one-line `## Install` hint with the canonical package + docs link.

## Quick detection probe

```bash
detect_wallets() {
  command -v bankr        >/dev/null && bankr whoami 2>/dev/null         && echo "✓ bankr authed"
  command -v circle       >/dev/null && circle wallet list --type agent --chain BASE --output json 2>/dev/null | head -1 && echo "✓ circle authed"
  command -v lobstercash  >/dev/null && lobstercash status 2>/dev/null   && echo "✓ lobstercash status above"
  command -v purl         >/dev/null && purl wallet list 2>/dev/null     && echo "✓ purl authed"
  command -v tempo        >/dev/null && tempo wallet whoami 2>/dev/null  && echo "✓ tempo authed"
  command -v onchainos    >/dev/null && echo "✓ onchainos installed (signer-only; check balance via OKX dashboard)"
  command -v moonpay      >/dev/null && moonpay wallet list 2>/dev/null  && echo "✓ moonpay wallets listed above"
  npx --no-install awal status 2>/dev/null      && echo "✓ awal authed"
  npx --no-install agentcash accounts 2>/dev/null && echo "✓ agentcash authed"
  npx --no-install @solana/pay account list 2>/dev/null && echo "✓ pay.sh authed (balance shown above)"
}
```

The probe is best-effort. Some wallets (onchainos) are signer-only and have no native auth/balance command; check off-CLI.

## Wallet catalog

All `USDC balance` columns return just the USDC amount on the relevant chain (filtered output) — see each per-wallet doc for the exact filter.

Each per-wallet doc carries the authoritative `Protocols & chains` list and `Install` hint.

| Wallet | Agentic chains | Protocols | Auth check | USDC balance | Install | Doc |
|---|---|---|---|---|---|---|
| bankr | Base | x402 | `bankr whoami` | `bankr wallet portfolio --chain base --json` + `jq` USDC filter | `npm i -g bankr` | [./bankr.md](./wallets/bankr.md) |
| circle | Base | x402 | `circle wallet list --type agent --chain BASE --output json` | off-CLI — Base RPC for USDC contract | `npm i -g @circle-fin/cli` | [./circle.md](./wallets/circle.md) |
| lobstercash | Base | x402 | `lobstercash status` | parse `Balance: ... usdc` from `lobstercash status` | `npm i -g lobstercash` | [./lobstercash.md](./wallets/lobstercash.md) |
| awal | Base, Solana | x402 | `awal status` | `awal balance` (filter to USDC line) | none — `npx awal@latest` | [./awal.md](./wallets/awal.md) |
| purl | Base, Solana, Tempo | x402 (Base, Solana), MPP (Tempo) | `purl wallet list` | `purl balance` (filter to USDC line) | see purl docs | [./purl.md](./wallets/purl.md) |
| agentcash | Base, Solana, Tempo | x402 (Base, Solana), MPP (Tempo) | `npx agentcash@latest accounts` | `npx agentcash@latest balance` (filter to USDC line) | none — `npx agentcash@latest` | [./agentcash.md](./wallets/agentcash.md) |
| onchainos | Base | x402 (signer-only) | `onchainos --help` (install check) | off-CLI — Base RPC for USDC contract | `npm i -g onchainos` | [./onchainos.md](./wallets/onchainos.md) |
| tempo | Tempo | MPP | `tempo wallet whoami` | `tempo wallet whoami` (filter to USDC line) | Tempo CLI + `tempo add request` | [./tempo.md](./wallets/tempo.md) |
| moonpay | Solana | x402 | `moonpay user retrieve` (or `moonpay wallet list`) | `moonpay --json token balance list --wallet <name> --chain solana` + `jq '.items[] \| select(.symbol=="USDC") \| .balance.amount'` | `npm i -g @moonpay/cli` | [./moonpay.md](./wallets/moonpay.md) |
| pay.sh | Solana | x402 | `npx @solana/pay account list` | same call — USDC shown directly | none — `npx @solana/pay` | [./pay-sh.md](./wallets/pay-sh.md) |

## Worked example — paying moltycash via `purl`

moltycash (the gig / tip / hire network at `api.molty.cash`) is a public x402 + MPP endpoint that's handy to copy-paste against. Using `purl` (auto-detects x402 vs MPP):

```bash
purl https://api.molty.cash/a2a -X POST \
  --json '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}'
```

Each per-wallet doc carries the equivalent invocation. For moltycash-specific endpoints, fees, and the `tip` / `hire` payloads, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).

## Per-wallet docs

Each doc has the same shape: frontmatter, description, `## Protocols & chains`, `## Install`, `## Detection`, `## Transport`, `## Notes`.

Fetchable directly when this skill is hosted:

```bash
curl https://<host>/agentic-wallets/wallets/<wallet>.md
```
