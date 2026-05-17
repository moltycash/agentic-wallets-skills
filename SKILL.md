---
name: agentic-wallets
description: Detect installed/authenticated wallet CLIs and send x402 / MPP paid HTTP requests via them. Covers 10 wallets across Base, Solana, and Tempo.
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

This is a fetchable skill — an LLM coding agent (Claude Code, Codex, Cursor, etc.) reads SKILL.md once and reuses it across any paid HTTP endpoint.

### Wallet selection — ask the human first

**Before any payment call, ask the human you are working for which wallet to use.** Do not auto-detect, do not default to env vars. Wallet selection is a deliberate choice; the human almost always has a preferred wallet.

> Skip the ask only if the human's original prompt already names a wallet (e.g. "using tempo CLI", "with bankr"). Use what they specified.

If the human asks you to **scan their system** to find what's installed, run the `detect_wallets` probe in the next section. Treat detection as **opt-in**, not as Step 1 of every call.

### The flow per call

1. **Ask the human** which wallet to use (or scan if they asked you to).
2. **Match the chain.** The paid endpoint's 402 response lists accepted chains in `accepts[].network`. The chosen wallet's `Protocols & chains` must intersect that set, and the wallet must hold enough USDC for `amount + fee`.
3. **Fetch the chosen wallet's transport doc.**
   ```bash
   curl https://molty.cash/skills/agentic-wallets/wallets/<wallet>.md
   ```
   `<wallet>` ∈ `bankr`, `circle`, `lobstercash`, `awal`, `purl`, `agentcash`, `onchainos`, `tempo`, `moonpay`, `pay-sh`. Each doc gives the exact CLI invocation pattern for that wallet's x402 or MPP transport.
4. **Combine wallet transport + endpoint payload.** The wallet doc tells you *how to send* (signing, headers, options). The paid endpoint's own spec tells you *what to send* (URL, request body, fee structure for any per-call payment cap). The [worked example](#worked-example--calling-a-paid-endpoint-via-purl) below shows the pattern with moltycash as the endpoint.

If no catalog wallet works for the human, every per-wallet doc carries a one-line `## Install` hint with the canonical package + docs link — offer a short menu of install options instead of guessing.

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

## Worked example — calling a paid endpoint via `purl`

This catalog tells you how to *use* a wallet. The paid endpoint you're calling tells you *what to send*:

- the endpoint URL
- the request body (e.g. JSON-RPC payload)
- the fee structure (needed for wallets like `bankr` that take a per-call `--max-payment` cap — pass `amount + fee + headroom`)

Concrete example: moltycash (the public tip / hire / gig network at `api.molty.cash`) publishes those details at [`https://molty.cash/skills/PAYMENT.md`](https://molty.cash/skills/PAYMENT.md). Fetching it gives you:

- **Endpoints** — `POST https://api.molty.cash/{username}/a2a` for tip/hire, `POST https://api.molty.cash/a2a` for `gig.create`
- **Payloads** — JSON-RPC 2.0, e.g. `{"jsonrpc":"2.0","id":1,"method":"tip","params":{"amount":0.50}}`
- **Fees** — flat **1¢** on payments under $1, **3%** on payments ≥ $1

Plug those into `purl` (which auto-detects x402 vs MPP per call):

```bash
# Tip 0xmesuthere 50¢ (total cost: 50¢ tip + 1¢ flat fee = 51¢)
purl https://api.molty.cash/0xmesuthere/a2a -X POST \
  --json '{"jsonrpc":"2.0","id":1,"method":"tip","params":{"amount":0.50}}'
```

Same call with `bankr` — note the explicit cap that includes the fee + headroom:

```bash
bankr x402 call https://api.molty.cash/0xmesuthere/a2a \
  --method POST --max-payment 0.60 \
  --body '{"jsonrpc":"2.0","id":1,"method":"tip","params":{"amount":0.50}}'
```

Each per-wallet doc carries the equivalent invocation for the other CLIs in the catalog. **Any other paid HTTP endpoint follows the same pattern** — read its own payment spec for URL + body + fees, then plug those into any catalog wallet's transport.

## Per-wallet docs

Each doc has the same shape: frontmatter, description, `## Protocols & chains`, `## Install`, `## Detection`, `## Transport`, `## Notes`.

Fetchable directly when this skill is hosted:

```bash
curl https://<host>/agentic-wallets/wallets/<wallet>.md
```
