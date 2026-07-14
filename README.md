# DAY Maker

**Solana market-making API + MCP for agents.**

| | |
|--|--|
| **MCP** | `https://dayprotocol.com/mm/mcp` (open — no unlock token) |
| **REST** | `https://dayprotocol.com/mm/v1` |
| **Telegram** | **[@daymmbot](https://t.me/daymmbot)** — full Maker MCP in chat |
| **Host** | **day-protocol** (dayprotocol.com) |
| **Billing** | **x402 on** — **$0.001 USDC** / billable call · discovery free |

## Wallet modes

| Mode | Keys | You do | Maker does |
|--|--|--|--|
| **[External (BYO)](./WALLET-MODES.md#mode-a--external-bring-your-own-wallet)** | Stay with you | Sign txs | Build unsigned swap + relay |
| **[Custodial (workers)](./WALLET-MODES.md#mode-b--custodial-maker-workers)** | Encrypted vault / secret-db | Call API / confirm jobs | Fund, warmup, strategies, volume bots |

→ Full comparison: **[WALLET-MODES.md](./WALLET-MODES.md)**

## Telegram bot

Chat **[@daymmbot](https://t.me/daymmbot)** for the same Maker surface as MCP.

- **Free:** hellos, learning (`/overview`, what can you do, `/help`, `/fees`)
- **Paid:** market-making actions **$0.001 USDC**/msg from prepaid credits (SOL deposit)
- **Feedback:** `/feedback …` (free, logged server-side)

## Fees & usage stats

```bash
curl -s https://dayprotocol.com/mm/v1/fees | jq .
curl -s https://dayprotocol.com/mm/v1/rate-limits | jq .
curl -s https://dayprotocol.com/mm/v1/x402/pricing | jq .   # free:false, $0.001 USDC
curl -s https://dayprotocol.com/mm/v1/stats | jq .          # counts only, no key secrets
```

| Env | Live default | Effect |
|--|--|--|
| `X402_ENABLED` | **`true`** | USDC pay-per-successful billable API/MCP call |
| `X402_PRICE_USDC` | **`0.001`** | Per billable call (Solana USDC → treasury) |
| `RATE_LIMIT_PER_SEC` | **`1`** | Free tier rate limit |
| `PAID_RATE_LIMIT_PER_SEC` | **`10`** | With `x-maker-rate-tier: paid` or x402 |
| `TELEGRAM_MSG_FEE_USDC` | **`0.001`** | Per MM chat message (learning free) |

Billable REST:

```bash
curl -s -H "authorization: Bearer $KEY" -H "x-payment: $X402_PAYLOAD" \
  https://dayprotocol.com/mm/v1/account
```

Without `x-payment` on billable routes → **HTTP 402** + `accepts[]`.

## Docs

| Doc | |
|--|--|
| [Wallet modes](./WALLET-MODES.md) | External vs custodial |
| [Getting started](./GETTING-STARTED.md) | Key → fund → strategy (+ Telegram) |
| [API reference](./API.md) | REST + MCP tools |
| [Strategy templates](./TEMPLATES.md) | Catalog, venues |
| [Concepts](./CONCEPTS.md) | Jobs, ranges, private drip |

## Use the API directly

```bash
export MAKER_API=https://dayprotocol.com/mm/v1
export MAKER_MCP=https://dayprotocol.com/mm/mcp

# mint account key (free, once)
curl -s -X POST "$MAKER_API/api-keys" -H 'content-type: application/json' \
  -d '{"label":"my-agent"}'
# → save mk_live_… once

# catalog (free)
curl -s "$MAKER_API/capabilities" | jq .
```
