# DAY Maker

**Solana market-making API + MCP for agents.**

Maker lets AI agents and developers:

- Create custodial worker wallets (keys vaulted — never exported)
- Deposit once and **stagger-fund** workers privately over time
- Warm up wallets, run **40+ strategy templates**, volume bots, LP scale plans
- Get **full token intel** (market + holders + traders) and a ranked strategy campaign
- Operate everything over **HTTPS REST** or **open MCP**

| | |
|--|--|
| **MCP** | `https://dayprotocol.com/mm/mcp` (open — no unlock token) |
| **REST** | `https://dayprotocol.com/mm/v1` |
| **Billing** | API/MCP calls are **free** right now |

## Docs

| Doc | |
|--|--|
| [Getting started](./GETTING-STARTED.md) | Create a key, fund, run a strategy |
| [API reference](./API.md) | REST endpoints + MCP tools |
| [Strategy templates](./TEMPLATES.md) | Full catalog with venues + best-for |
| [Concepts](./CONCEPTS.md) | Custody, jobs, ranges, private funding |

## Quick start

```bash
# 1) Mint API key (shown once — save it)
curl -s -X POST https://dayprotocol.com/mm/v1/api-keys \
  -H 'content-type: application/json' \
  -d '{"label":"my-agent"}'

# 2) List everything you can call
curl -s https://dayprotocol.com/mm/v1/capabilities | jq '.api_calls | length'

# 3) Strategy guide + templates
curl -s https://dayprotocol.com/mm/v1/strategy-templates | jq '.templates | length'
```

MCP clients: connect to `https://dayprotocol.com/mm/mcp` and pass `api_key` (`mk_live_…`) on org-scoped tools.

## Custody model

**Default = custodial.** Maker mints worker keypairs and stores them encrypted. You operate via API/MCP only.  
**No private key export.** Cash out with `maker_wallet_withdraw` / `POST /v1/wallets/withdraw` to *your* address.

Optional **BYO / external wallets**: register a pubkey, build an unsigned swap, you sign client-side.

## Telegram

`@daymmbot` — same MCP tool surface, API key auto-linked on first DM, confirm-before-execute for money moves.

## Status

Generated from production capabilities · 2026-07-13 UTC
