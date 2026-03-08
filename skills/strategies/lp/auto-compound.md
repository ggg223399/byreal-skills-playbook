# Strategy #9: Auto Compound

> Automatically reinvest accrued LP fees back into the position to compound returns.

## Overview

| Attribute | Value |
|-----------|-------|
| Category | LP — Yield Optimization |
| Risk | Conservative (C) |
| Complexity | * Single-step |
| Market | All (*) |
| Cycle | D |
| Min Capital | 500 USDC |

## Requires

```yaml
feeds:
  - pool-state       # current price, pool metrics
  - position         # unclaimed fees, current range, IL

execution:
  - collect-fees
  - swap
  - add-liquidity

risk:
  - il-guard
  - gas-check
```

## Parameters

| Param | Type | Default | Constraints | Description |
|-------|------|---------|-------------|-------------|
| `pool` | string | — | required | Pool address or token pair |
| `position_id` | string | — | required (or auto-detect) | Position to compound |
| `compound_threshold` | USDC | 10 | 1-1000 | Minimum unclaimed fees (in USDC value) to trigger compound |
| `compound_mode` | enum | same_range | same_range / new_position | Reinvest into same range or create new position |
| `check_interval` | duration | 6h | 1h-24h | How often to check unclaimed fees |
| `il_pause_threshold` | percent | 5% | 1%-20% | Pause compounding if IL exceeds this |
| `slippage_tolerance` | percent | 1% | 0.1%-5% | Max slippage for fee token swap |

## Decision Logic

```
EVERY check_interval:

1. FETCH position → { unclaimed_fee_a, unclaimed_fee_b, lower_tick, upper_tick, il_percent }
2. CALCULATE total_fees_usd = unclaimed_fee_a * price_a + unclaimed_fee_b * price_b

3. IF total_fees_usd < compound_threshold:
     → HOLD (fees too small, wait for more to accumulate)

4. IF il_percent > il_pause_threshold:
     → HOLD + WARN "Compounding paused: IL at {il_percent}%, threshold is {il_pause_threshold}%"

5. CHECK gas-check → if gas > 10% of total_fees_usd, HOLD (not worth it)

6. EXECUTE compound:
     a. collect-fees(position_id) → receive fee_token_a, fee_token_b

     b. FETCH pool_state → current_price
     c. CALCULATE target_ratio for current range [lower_tick, upper_tick]
     d. CALCULATE current_ratio from fee_token_a, fee_token_b amounts

     e. IF current_ratio != target_ratio:
          swap(excess_token → deficit_token, slippage_tolerance)

     f. IF compound_mode == "same_range":
          add-liquidity(pool, lower_tick, upper_tick, token_a, token_b)
          → increases liquidity in existing range

        IF compound_mode == "new_position":
          → create a separate position (useful for tracking compound performance)

7. LOG compound event: fees_collected, fees_reinvested, swap_cost, new_total_liquidity
```

## Exit Conditions

| Condition | Action |
|-----------|--------|
| IL exceeds `il_pause_threshold` | Pause compounding, alert user |
| IL exceeds `il-guard` hard limit | Alert user, suggest full exit |
| Fees consistently below threshold for 7+ days | Alert user: "Low fee generation, consider adjusting range or migrating" |
| User requests stop | Stop compounding, position remains open |

## Performance Metrics

Track and report:
- Total fees collected (cumulative)
- Total fees reinvested
- Compound count
- Compound cost (swap fees + gas)
- Effective APY vs simple APR (shows compounding benefit)
- Current position value vs initial deposit

## Interaction with Other Strategies

Auto Compound works well as a **secondary module** attached to other strategies:

| Primary Strategy | Combo Behavior |
|-----------------|----------------|
| #1 Auto Range Rebalance | Compound between rebalances; on rebalance, fees are collected as part of remove-liquidity |
| #4 Multi-Range Split | Compound each sub-position independently |
| #13 APR Target | Compound increases liquidity, which may affect APR target calculations |

When used as a module, set `compound_mode: same_range` and let the primary strategy handle range decisions.

## Example Configurations

### Passive (low-frequency, high threshold)
```
pool: SOL/USDC
compound_threshold: 50
check_interval: 24h
il_pause_threshold: 5%
```

### Active (frequent compounding)
```
pool: SOL/USDC
compound_threshold: 5
check_interval: 2h
il_pause_threshold: 8%
```
