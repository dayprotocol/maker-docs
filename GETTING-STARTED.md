# Maker — Getting started

**Maker** is DAY Protocol’s Solana market-making API for agents.  
Use **HTTPS REST** or **open MCP** with a long-lived API key (`mk_live_…`).

| | |
|--|--|
| MCP | `https://dayprotocol.com/mm/mcp` (no unlock token) |
| REST | `https://dayprotocol.com/mm/v1` |
| Telegram | **[@daymmbot](https://t.me/daymmbot)** |
| Concepts | [CONCEPTS.md](./CONCEPTS.md) |
| Templates | [TEMPLATES.md](./TEMPLATES.md) |
| Full API | [API.md](./API.md) |

**Wallet modes:** Maker supports **external (you sign)** and **custodial (Maker workers)**.  
Full comparison → **[WALLET-MODES.md](./WALLET-MODES.md)**.

| Mode | Keys | Start here |
|--|--|--|
| **External** | Stay with you | [§ External path](#external-path-i-only-sign) |
| **Custodial** | Vaulted by Maker | [§ Custodial path](#3-create-worker-wallets) below |

**Billing:** API, MCP, and Telegram are **FREE right now** (no deposit, no per-call fee).  
Optional later: USDC fees via `X402_ENABLED` / `TELEGRAM_MSG_FEE_USDC` — see `GET /v1/fees`.

**Telegram:** DM **[@daymmbot](https://t.me/daymmbot)** — it mints your org + API key on first chat and runs the full Maker tool catalog (confirm before mutates).

---

## What you can do

**Either mode**

1. **Mint an API key** → that key *is* your account (not a chain key)  
2. **Token intel + campaign** — market, holders, traders → ranked strategy ideas  
3. **List / cancel / poll jobs**  

**External (BYO)**

4. Register your pubkey → build unsigned swap → **you sign** → submit  

**Custodial**

5. Open funding session → deposit SOL once  
6. Create workers → staggered private fund drip  
7. Warmup + strategies / volume bots  
8. Live-edit jobs · withdraw to *your* address (keys never exported)

---

## 1. Create an API key

```bash
export MAKER_API=https://dayprotocol.com/mm/v1

curl -s -X POST "$MAKER_API/api-keys" \
  -H 'content-type: application/json' \
  -d '{"label":"friend-test"}' | tee /tmp/maker-key.json
```

Save:

- `key` — `mk_live_…` (**shown once** — store in a password manager)  
- `org_id`  
- `prefix` — public short id  

```bash
export MAKER_KEY=$(jq -r .key /tmp/maker-key.json)
```

Auth header on later calls:

```bash
-H "Authorization: Bearer $MAKER_KEY"
# or
-H "x-api-key: $MAKER_KEY"
```

If you lose the key, mint another under the same org (with a still-valid key) or start a new org. There is no recovery of the full secret.

---

## External path (“I only sign”)

No deposit required for trading your own wallet. Maker never sees your private key.

```bash
# Register pubkey
curl -s -X POST "$MAKER_API/wallets/external" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{"pubkey":"<YOUR_PUBKEY>","label":"main"}' | jq .

# Build unsigned swap
curl -s -X POST "$MAKER_API/trades/build" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{
    "pubkey": "<YOUR_PUBKEY>",
    "input_mint": "So11111111111111111111111111111111111111112",
    "output_mint": "<TOKEN_MINT>",
    "amount_sol": 0.05
  }' | jq .
# → unsigned_tx_base64, trade_id  — sign locally, then:

curl -s -X POST "$MAKER_API/trades/<trade_id>/submit" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{"signed_tx_base64":"<SIGNED>"}' | jq .
```

MCP: `maker_external_wallet_register` → `maker_trade_build` → (sign) → `maker_trade_submit`.

Details: [WALLET-MODES.md](./WALLET-MODES.md).

---

## 2. Funding session (deposit once) — custodial

```bash
curl -s -X POST "$MAKER_API/funding/sessions" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{}' | jq .
```

- `deposit_address` — **send SOL here** (mainnet)  
- Balance becomes `available_sol` for ops  

Check balance:

```bash
curl -s "$MAKER_API/funding/balance" \
  -H "Authorization: Bearer $MAKER_KEY" | jq .
```

---

## 3. Create worker wallets

```bash
curl -s -X POST "$MAKER_API/wallets/batch" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{
    "count": 3,
    "create_per_day": {"min": 2, "max": 3},
    "create_spacing_minutes": {"min": 120, "max": 300},
    "name": "Create 3 workers · paced"
  }' | jq .
```

Poll the returned job until wallets exist:

```bash
curl -s "$MAKER_API/jobs?kind=create_wallets" \
  -H "Authorization: Bearer $MAKER_KEY" | jq .
```

---

## 4. Fund workers (staggered private drip — default)

**Default behavior:** one deposit → private withdraws to many workers over time with:

- **Random amounts** (`amount_range_sol`, default ~0.08–0.45 SOL)  
- **Random gaps** (`fund_spacing_minutes`, default **1 minute – 24 hours**)  
- Shuffled order + asymmetric shield vs withdraw  

```bash
curl -s -X POST "$MAKER_API/funding/allocate" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{
    "wallet_ids": "all_unfunded",
    "amount_range_sol": {"min": 0.08, "max": 0.45, "unit": "sol"},
    "fund_spacing_minutes": {"min": 1, "max": 1440},
    "name": "Drip-fund workers · 1m–24h gaps"
  }' | jq .
```

Speed up later without recreating the job:

```bash
curl -s -X PATCH "$MAKER_API/jobs/<job_id>" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{"speed":"fast"}'   # 1–15m gaps + run now
```

---

## 5. Pick a strategy (full intel campaign)

```bash
# Best decision path: market + holders + traders → score all templates
curl -s "$MAKER_API/strategies/campaign?contract=<MINT>&top_n=5" | jq '{summary, execute_order, parallel_groups}'
```

Or quick regime recommend:

```bash
curl -s "$MAKER_API/strategies/recommend?contract=<MINT>" | jq .
```

List templates:

```bash
curl -s "$MAKER_API/strategy-templates" | jq '.templates[] | {id, name, best_for, venues}'
```

---

## 6. Create + start a strategy

```bash
# wallet_ids from GET /wallets
curl -s -X POST "$MAKER_API/strategies" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{
    "token_mint": "<MINT>",
    "wallet_ids": ["wal_…", "wal_…"],
    "template": "balanced_pmm",
    "name": "balanced_pmm · mint… · 3w"
  }' | jq .

curl -s -X POST "$MAKER_API/strategies/<strategy_id>/start" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{}' | jq .
```

### Volume bot modes (multi-maker)

| Template | Mode |
|--|--|
| `volume_boost` | Steady multi-wallet buy/sell, random delays |
| `volume_bump` | Buy-heavy until budget, then rebalance sells |
| `volume_advanced` | Full knobs: makers, ratio, delay, volume target |

---

## 7. Live-edit jobs

```bash
# Speed up fund drip
curl -s -X PATCH "$MAKER_API/jobs/<job_id>" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{"fund_spacing_minutes":{"min":5,"max":30},"run_now":true}'

# Retune strategy knobs via strategy tick job or:
curl -s -X PATCH "$MAKER_API/strategies/<strategy_id>" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{"acts_per_hour":{"min":10,"max":30},"volume_mode":"boost"}'
```

---

## 8. Withdraw (cash out)

```bash
curl -s -X POST "$MAKER_API/wallets/withdraw" \
  -H "Authorization: Bearer $MAKER_KEY" \
  -H 'content-type: application/json' \
  -d '{
    "wallet_id": "wal_…",
    "to_address": "<YOUR_PUBKEY>",
    "amount": {"min": 0.1, "max": 0.1, "unit": "sol"}
  }' | jq .
```

Keys stay vaulted — only SOL moves to your address.

---

## MCP

- Endpoint: **`https://dayprotocol.com/mm/mcp`** (open)  
- Org tools need `api_key` in args (or Telegram auto-injects it)  
- Useful tools: `maker_capabilities`, `maker_strategy_campaign`, `maker_plan` / `maker_execute_plan`, `maker_job_patch`, `maker_lp_scale_plan`

---

## Ranges

Almost every tunable is `{ "min": n, "max": n, "unit?": "sol|minutes|hours|days" }`.  
Pin `min === max` for a fixed value. Server samples within the band for organic-looking activity.

---

## Rules of thumb

| | |
|--|--|
| API key | Shown once — save it |
| Custody | We mint+vault keys; **no key export** |
| Funding | Deposit once → staggered private drip (default) |
| Amounts / times | Always ranges unless you pin min=max |
| Jobs | Every action is a job — poll status / activity |
| Edit later | `PATCH /jobs/:id` or `maker_job_patch` |
| Billing | Free for now |

Questions → open an issue on this repo or use Telegram `@daymmbot`.
