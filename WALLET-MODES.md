# Wallet modes

Maker supports **two wallet modes**. Pick based on how you want to operate — not a ranking of “more secure,” just different tradeoffs.

| | **External (BYO)** | **Custodial (Maker workers)** |
|--|--|--|
| **Who holds the private key** | You | Maker (encrypted vault) |
| **How trades happen** | Maker builds **unsigned** tx → **you sign** → Maker submits | Maker signs with vaulted worker keys |
| **Best for** | “I only sign” · agents/apps that already have a wallet · max key control | Full automation: fund drip, warmup, strategies, volume bots, multi-wallet jobs |
| **Setup** | Register pubkey only | Create workers via API/MCP |
| **Funding** | You fund **your** wallet yourself | Deposit once → staggered private drip to workers |
| **Cash out** | You already control the wallet | `withdraw` from worker → **your** address (keys never exported) |
| **API key** | Still need `mk_live_…` (account auth, **not** a chain key) | Same |

---

## Mode A — External (bring your own wallet)

**You keep the private key. Maker never receives, stores, or exports it.**

### Flow

1. **Register** pubkey  
2. **Build** unsigned swap  
3. **Sign** locally (wallet / agent / SDK)  
4. **Submit** signed tx (must match what Maker built; ~60s blockhash window)

### MCP

```
maker_external_wallet_register  →  pubkey only
maker_trade_build               →  unsigned_tx_base64 + trade_id
# you sign client-side
maker_trade_submit              →  signed_tx_base64
maker_trades_list / maker_trade_get
```

### REST

```http
POST /v1/wallets/external
POST /v1/trades/build
POST /v1/trades/:id/submit
GET  /v1/trades
GET  /v1/trades/:id
```

### What external mode does **not** include

Custodial-only automation:

- Private fund drip to workers  
- Warmup  
- Strategy engine / volume boost·bump·advanced  
- Withdraw-from-vault  

Those need **custodial workers**. External wallets are hard-blocked from those ops.

### When to use external

- You already have a funded wallet  
- Policy is “keys never leave my box”  
- Agent can sign locally  
- One-off or scripted swaps, not full MM fleets  

---

## Mode B — Custodial (Maker workers)

**Maker mints keypairs and stores them encrypted. You operate only via API/MCP.**

### Flow (typical)

1. Create API key (`mk_live_…`)  
2. Open funding session → deposit **SOL once**  
3. Batch-create workers  
4. Allocate (default: **staggered private drip**, random amounts, gaps 1m–24h)  
5. Optional warmup  
6. Strategy create/start (or volume templates)  
7. Withdraw to **your** address when you want SOL out  

**Private keys are never exported or downloaded.** Cash-out is withdraw-to-address only.

### MCP (highlights)

```
maker_wallets_batch
maker_funding_session / maker_funding_allocate
maker_warmup
maker_strategy_create / maker_strategy_start
maker_strategy_campaign / maker_token_intel
maker_job_patch          → live-edit speed/randomness
maker_wallet_withdraw    → to your pubkey
```

### When to use custodial

- You want **unattended** multi-wallet jobs  
- Volume boost / bump / advanced  
- Fund drip, warmup, long-running strategies  
- Agents that shouldn’t hold chain keys  

---

## Same account, both modes

One `mk_live_…` org can:

- Register **external** wallets for sign-only trades, **and**  
- Create **custodial** workers for automation  

They don’t mix on the same wallet id: external wallets can’t be used in custodial fund/warmup/strategy paths.

---

## Quick chooser

| You want… | Use |
|--|--|
| Keys stay with me; I only sign | **External** |
| Full MM / volume fleet on autopilot | **Custodial** |
| Agent with a browser/extension wallet | **External** |
| Agent that shouldn’t touch secrets | **Custodial** |
| Deposit once, many workers over time | **Custodial** fund drip |
| Inspect unsigned swap before sign | **External** (`trade_build`) |

---

## Related

- [Getting started](./GETTING-STARTED.md)  
- [Concepts](./CONCEPTS.md)  
- [API reference](./API.md)  
- [Strategy templates](./TEMPLATES.md)  

**MCP:** `https://dayprotocol.com/mm/mcp` · **REST:** `https://dayprotocol.com/mm/v1`
