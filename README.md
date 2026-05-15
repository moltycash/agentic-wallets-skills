# agentic-wallets-skills

Generic [x402](https://x402.org) + MPP (Multi-Provider Payments) wallet skill for AI agents.

A catalog of 10 wallet CLIs that can sign paid HTTP requests, with detection probes (auth + USDC balance) and per-chain transport patterns. The skill assumes a wallet is already installed and authed; if none are present, the AI agent decides which to install.

## Entry point

Point any agent at [`SKILL.md`](./SKILL.md). It contains:

- Detection probe for all 10 wallet CLIs (auth + USDC balance commands)
- Wallet catalog with `Protocols & chains`, install hints, and per-wallet doc links
- A worked example using [molty.cash](https://molty.cash) (a public x402 + MPP endpoint) for copy-pasteable testing

## Wallets covered

| Wallet | Chains | Protocols |
|---|---|---|
| [bankr](./bankr.md) | Base | x402 |
| [circle](./circle.md) | Base | x402 |
| [lobstercash](./lobstercash.md) | Base | x402 |
| [awal](./awal.md) | Base, Solana | x402 |
| [purl](./purl.md) | Base, Solana, Tempo | x402, MPP |
| [agentcash](./agentcash.md) | Base, Solana, Tempo | x402, MPP |
| [onchainos](./onchainos.md) | Base | x402 (signer-only) |
| [tempo](./tempo.md) | Tempo | MPP |
| [moonpay](./moonpay.md) | Solana | x402 |
| [pay.sh](./pay-sh.md) | Solana | x402 |

## Install / use the skill

This is an [agentskills.io](https://agentskills.io/specification)-style skill — a folder with `SKILL.md` at the root. There are four common ways to install it:

### 1. Direct fetch (zero-install)

Have your agent `curl` the skill at request time:

```bash
curl https://raw.githubusercontent.com/0xsnackbaker/agentic-wallets-skills/main/SKILL.md
```

Then fetch individual wallet docs as needed:

```bash
curl https://raw.githubusercontent.com/0xsnackbaker/agentic-wallets-skills/main/bankr.md
```

### 2. Claude Code skill

Drop the repo into your Claude Code skills folder:

```bash
git clone https://github.com/0xsnackbaker/agentic-wallets-skills.git \
  ~/.claude/skills/agentic-wallets
```

Claude Code picks up `SKILL.md` automatically and the docs become invokable.

### 3. Git submodule

Pin to a specific commit in your own repo:

```bash
git submodule add https://github.com/0xsnackbaker/agentic-wallets-skills.git skills/agentic-wallets
git submodule update --init
```

### 4. Reference from another skill

Cross-link from your own SKILL.md to this one:

```markdown
> Need to send a paid HTTP request? See the agentic-wallets skill at
> https://raw.githubusercontent.com/0xsnackbaker/agentic-wallets-skills/main/SKILL.md
```

## Skill shape

Each per-wallet doc follows a uniform structure so agents can parse them consistently:

```
---
name: wallet-<name>
description: <one-line>
license: MIT
---

# <wallet name>

## Install         — one-line install hint
## Protocols & chains  — x402 / MPP × chain list
## Detection      — auth check + USDC balance command
## Transport      — how to POST a body to any paid endpoint
## Example        — concrete copy-paste example
## Notes          — wallet-specific quirks (optional)
```

## License

MIT.

## Contributing

PRs welcome for new wallet integrations or corrections to existing ones. New wallets should follow the shape above and include verified `Detection` commands (don't fabricate CLI commands — capture them from the wallet's actual `--help` output).
