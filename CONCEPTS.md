# Maker concepts

## Custody

| Mode | Keys | Use |
|--|--|--|
| **Custodial (default)** | Maker mints + encrypts in vault | Fund, warmup, strategies, withdraw to your address |
| **External / BYO** | You keep keys | Register pubkey → build unsigned swap → you sign → submit |

Private keys are **never** returned by API or MCP.

## API key = account

- `POST /v1/api-keys` returns `mk_live_…` **once**
- Server stores hash + prefix only
- Pass on every org-scoped call: `Authorization: Bearer …`

## Jobs

Every long-running action is a **job** with:

- `name` (human label)
- `status` — scheduled | running | settling | completed | failed | cancelled  
- `progress` — `{ pct, done, total, message }`  
- `activity[]` — event trail  

Poll `GET /v1/jobs` / `GET /v1/jobs/:id` or MCP `maker_jobs_list` / `maker_job_status`.

**Live edit:** `PATCH /v1/jobs/:id` / `maker_job_patch` — change spacing, amounts, speed, strategy knobs after start.

## Ranges

```json
{ "min": 0.08, "max": 0.45, "unit": "sol" }
```

Server samples within the band for organic-looking activity. Pin min=max for fixed.

## Private funding (default drip)

1. Open funding session → deposit address  
2. Send SOL **once**  
3. `POST /v1/funding/allocate` drip-funds workers:
   - Random **amounts** (`amount_range_sol`)
   - Random **gaps** (`fund_spacing_minutes`, default **1 min – 24 hours**)
   - Shuffled order, optional asymmetric shield vs withdraw  

Override spacing anytime with job patch (`speed: "fast"` or custom range).

## Strategies

- **Templates** — 40+ presets (PMM, volume boost/bump/advanced, dip/TP, LP scale)  
- **Venues** — Meteora DLMM, Raydium CLMM/AMMv4, Orca Whirlpools, Jupiter routing  
- **Campaign** — `GET /v1/strategies/campaign?contract=` scores all templates from market + holders + traders  
- **LP scale** — `maker_lp_scale_plan` one-sided scale-in/out ladders  

## Volume modes

| Template | Behavior |
|--|--|
| `volume_boost` | Steady multi-maker buy/sell, random delays |
| `volume_bump` | Buy-heavy until budget, then rebalance sells |
| `volume_advanced` | Full knobs: makers, ratio, delay, volume target |

Uses custodial workers + on-chain swaps (Jito tip defaulted). Confirm before start.

## Orchestrator

1. `maker_plan` — freeform intent → multi-step plan + defaults  
2. User confirms (`go` / `yes`)  
3. `maker_execute_plan` — runs create → fund → warmup → strategy as planned  

## Security highlights

- No private key export  
- API secrets hashed at rest  
- Telegram: hard confirm on mutating tools; withdraw blocked on bot (use REST)  
- Remote access over HTTPS only  

## Related

- [GETTING-STARTED.md](./GETTING-STARTED.md)  
- [API.md](./API.md)  
- [TEMPLATES.md](./TEMPLATES.md)  
