# Strategy Catalog

## LP Strategies (Byreal CLMM) — 26 strategies

Concentrated liquidity strategies on Byreal DEX. Manage LP positions, optimize yield, and control impermanent loss.

### Range Management — Position control

| # | Strategy | Risk | Complexity | Market | Cycle | Min Capital | File |
|---|----------|------|------------|--------|-------|-------------|------|
| 1 | **Auto Range Rebalance** — Rebalance when price drifts out of range | C | ** | * | H-D | 1k | `lp/auto-range-rebalance.md` |
| 2 | **Volatility-Adaptive Range** — Widen/narrow range based on ATR | C | ** | * | H | 1k | `lp/volatility-adaptive.md` |
| 3 | **Narrow Range Sniper** — Tight range before volatility events | A | * | V | min-H | 5k | `lp/narrow-sniper.md` |
| 4 | **Multi-Range Split** — Deploy multiple ranges (core + wings) | M | ** | R | D | 3k | `lp/multi-range-split.md` |
| 5 | **Range Trailing** — Range center follows price like trailing stop | M | ** | T | H | 2k | `lp/range-trailing.md` |
| 6 | **Mean-Reversion Range** — Hold range when price deviates, earn on reversion | M | ** | R | D | 2k | `lp/mean-reversion-range.md` |
| 7 | **Breakout Range** — Deploy at new levels after breakout | A | ** | T | H | 3k | `lp/breakout-range.md` |
| 8 | **Single-Sided LP** — Provide one side only, wait for price to enter | M | * | T | D | 1k | `lp/single-sided.md` |

### Yield Optimization — Maximize LP returns

| # | Strategy | Risk | Complexity | Market | Cycle | Min Capital | File |
|---|----------|------|------------|--------|-------|-------------|------|
| 9 | **Auto Compound** — Reinvest fees when they exceed threshold | C | * | * | D | 500 | `lp/auto-compound.md` |
| 10 | **Incentive Harvester** — Claim & reinvest incentive rewards | C | * | * | D-W | 500 | `lp/incentive-harvester.md` |
| 11 | **Fee Tier Optimizer** — Migrate to higher-APR fee tier | C | ** | * | W | 2k | `lp/fee-tier-optimizer.md` |
| 12 | **Pool Rotation** — Rotate capital to top-N APR pools | M | ** | * | D-W | 3k | `lp/pool-rotation.md` |
| 13 | **APR Target** — Auto-adjust range width to hit target APR | M | ** | * | D | 2k | `lp/apr-target.md` |
| 14 | **Incentive + Fee Max** — Optimize for combined fee + incentive APR | M | ** | * | D | 2k | `lp/incentive-fee-max.md` |
| 15 | **JIT Liquidity** — Just-in-time liquidity for large swaps | A | *** | * | min | 10k | `lp/jit-liquidity.md` |

### Lifecycle Management — Entry, exit, migration

| # | Strategy | Risk | Complexity | Market | Cycle | Min Capital | File |
|---|----------|------|------------|--------|-------|-------------|------|
| 16 | **LP DCA** — Gradual entry in N batches | C | * | * | D-W | 1k | `lp/lp-dca.md` |
| 17 | **LP Gradual Exit** — Split exit to minimize slippage | C | * | * | D | 1k | `lp/lp-gradual-exit.md` |
| 18 | **IL Monitor + Auto Exit** — Exit when IL exceeds threshold | C | * | V | H | 500 | `lp/il-monitor-auto-exit.md` |
| 19 | **LP Position Aging** — Narrow range early, widen as position ages | M | ** | * | W | 2k | `lp/position-aging.md` |
| 20 | **LP Migration** — Migrate between pools on incentive changes | C | ** | * | W | 1k | `lp/lp-migration.md` |

### CopyFarmer Enhanced

| # | Strategy | Risk | Complexity | Market | Cycle | Min Capital | File |
|---|----------|------|------------|--------|-------|-------------|------|
| 21 | **Smart Copy** — Copy top farmer + Perp delta hedge | M | *** | * | D | 3k | `lp/smart-copy.md` |
| 22 | **Selective Copy** — Copy only specific pool/token operations | M | ** | * | D | 1k | `lp/selective-copy.md` |
| 23 | **Copy with Range Mod** — Copy farmer center, custom width | M | ** | * | D | 1k | `lp/copy-range-mod.md` |
| 24 | **Multi-Farmer Blend** — Weighted blend of multiple farmers | M | ** | * | D | 5k | `lp/multi-farmer-blend.md` |
| 25 | **Farmer Alpha Tracking** — Alert on anomalous farmer actions | C | * | * | H | 1k | `lp/farmer-alpha-tracking.md` |
| 26 | **Anti-Copy** — Only copy high-winrate operations | M | ** | * | D | 2k | `lp/anti-copy.md` |

---

## Legend

| Tag | Meaning |
|-----|---------|
| **Risk** | `C` Conservative / `M` Moderate / `A` Aggressive |
| **Complexity** | `*` Single-step / `**` Multi-step / `***` Multi-leg |
| **Market** | `T` Trending / `R` Ranging / `V` High-vol / `Q` Low-vol / `*` All |
| **Cycle** | `min` Minutes / `H` Hours / `D` Days / `W` Weeks |
| **Min Capital** | Recommended starting capital in USDC |
