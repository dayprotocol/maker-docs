# DAY Maker

**Solana market-making API + MCP for agents.**

| | |
|--|--|
| **MCP** | `https://dayprotocol.com/mm/mcp` (open — no unlock token) |
| **REST** | `https://dayprotocol.com/mm/v1` |
| **Billing** | API/MCP calls are **free** right now |

## Wallet modes

Maker has **two wallet modes** (pick what fits — not a ranking):

| Mode | Keys | You do | Maker does |
|--|--|--|--|
| **[External (BYO)](./WALLET-MODES.md#mode-a--external-bring-your-own-wallet)** | Stay with you | Sign txs | Build unsigned swap + relay |
| **[Custodial (workers)](./WALLET-MODES.md#mode-b--custodial-maker-workers)** | Encrypted vault | Call API / confirm jobs | Fund, warmup, strategies, volume bots |

→ Full comparison: **[WALLET-MODES.md](./WALLET-MODES.md)**

## Docs

| Doc | |
|--|--|
| [Wallet modes](./WALLET-MODES.md) | External vs custodial — how each works |
| [Getting started](./GETTING-STARTED.md) | Key → fund → strategy |
| [API reference](./API.md) | REST + MCP tools |
| [Strategy templates](./TEMPLATES.md) | Catalog, venues, best-for |
| [Concepts](./CONCEPTS.md) | Jobs, ranges, private drip, orchestrator |

## Quick start

```bash
# 1) Mint API key (shown once — save it)
curl -s -X POST https://dayprotocol.com/mm/v1/api-keys \
  -H 'content-type: application/json' \
  -d '{"label":"my-agent"}'

# 2) External mode — register your pubkey (you sign later)
curl -s -X POST https://dayprotocol.com/mm/v1/wallets/external \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{"pubkey":"<YOUR_PUBKEY>"}'

# 3) Or list capabilities / templates
curl -s https://dayprotocol.com/mm/v1/capabilities | jq '.api_calls | length'
curl -s https://dayprotocol.com/mm/v1/strategy-templates | jq '.templates | length'
```

MCP: connect to `https://dayprotocol.com/mm/mcp`. Pass `api_key` (`mk_live_…`) on org-scoped tools.

## What Maker can do

- **External trades** — build unsigned swap → you sign → submit  
- **Custodial workers** — mint, vault, operate via API only (no key export)  
- **Deposit once → staggered private fund drip** (random amounts, 1m–24h gaps)  
- **40+ strategy templates** (PMM, volume boost/bump/advanced, dip/TP, LP scale)  
- **Token intel + campaign** — market, holders, traders → ranked plan  
- **Live job edit** — speed up drip, retune randomness after start  
- **Telegram** `@daymmbot` — same tools, confirm-before-execute on money moves  

## Custody in one line

- **External:** private key never touches Maker.  
- **Custodial:** Maker holds worker keys in vault; **withdraw to your address only** — keys are never exported.

## Status

Public docs for the hosted API · [dayprotocol/maker-docs](https://github.com/dayprotocol/maker-docs)
