---
name: agentic-wallets
description: Pay any x402 (HTTP 402) or MPP (Multi-Provider Payments) HTTP endpoint by using one of 10 installed wallet CLIs (bankr, circle, lobstercash, awal, purl, agentcash, onchainos, tempo, moonpay, pay.sh). Covers Base, Solana, and Tempo.
license: MIT
metadata:
  author: molty.cash
  version: "3.0.0"
---

# agentic-wallets

A catalog of wallet CLIs that can sign **x402** (HTTP 402) or **MPP** (Multi-Provider Payments) paid HTTP requests. Use this to pay any paid endpoint with one of the wallets the human you're working for has already set up.

## Protocols & chains supported

- **x402**: Base (`eip155:8453`), Solana (`solana:5eykt4...`)
- **MPP**: Tempo (`eip155:4217`)

## How to use

### 1. Ask the human which wallet to use

**Before any payment call, ask the human you are working for which wallet to use.** Do not auto-detect, do not default to `*_PRIVATE_KEY` env vars. Wallet selection is a deliberate choice; the human almost always has a preferred wallet.

> Skip the ask only if **either**:
> - the human's original prompt already named a wallet (e.g. "using tempo CLI", "with bankr") — use what they specified, or
> - the agent has only one default wallet configured / authed for the relevant chain — use it.

If the human asks you to **scan their system** to find what's installed, run the `detect_wallets` probe in the next section. Treat detection as **opt-in**, not as a default step.

### 2. Match the chain

The paid endpoint's 402 response lists accepted chains in `accepts[].network`. The chosen wallet's `Protocols & chains` must intersect that set, and the wallet must hold enough USDC for `amount + fee`.

### 3. Fetch the chosen wallet's doc

```bash
curl https://molty.cash/skills/agentic-wallets/wallets/<wallet>.md
```

`<wallet>` ∈ `bankr`, `circle`, `lobstercash`, `awal`, `purl`, `agentcash`, `onchainos`, `tempo`, `moonpay`, `pay-sh`. Each doc has the exact CLI invocation pattern for that wallet's x402 or MPP transport.

### 4. Combine wallet transport + endpoint payload

The wallet doc tells you *how to send* (signing, headers, options). The paid endpoint's own spec tells you *what to send* (URL, request body, fees, per-call payment cap if any). Substitute and call.

If no catalog wallet works for the human, every per-wallet doc carries a one-line `## Install` hint — offer a short menu of install options instead of guessing.

## Quick detection probe (opt-in)

Only run this when the human asks you to scan.

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

---

Hosted at <https://molty.cash/skills/agentic-wallets/SKILL.md>.
