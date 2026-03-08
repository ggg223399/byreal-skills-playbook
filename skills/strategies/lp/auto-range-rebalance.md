# Strategy #1: Auto Range Rebalance

> Automatically rebalance LP position when price drifts out of range.

## Overview

| Attribute | Value |
|-----------|-------|
| Category | LP — Range Management |
| Risk | Conservative (C) |
| Complexity | ** Multi-step |
| Market | All (*) |
| Cycle | H-D |
| Min Capital | 1,000 USDC |

## Requires

```yaml
feeds:
  - pool-state        # current price, tick, liquidity
  - position          # current range, unclaimed fees, IL
  - market-indicators  # ATR for dynamic range width

execution:
  - remove-liquidity
  - add-liquidity
  - swap

risk:
  - il-guard
  - gas-check
  - rebalance-cooldown
```

## Parameters

| Param | Type | Default | Constraints | Description |
|-------|------|---------|-------------|-------------|
| `pool` | string | — | required | Pool address or token pair (e.g., "SOL/USDC") |
| `drift_threshold` | percent | 5% | 1%-20% | Price deviation from range center to trigger rebalance |
| `range_width` | percent | 10% | 2%-50% | Total width of LP range as % of current price |
| `range_mode` | enum | fixed | fixed / atr_adaptive | Fixed width or ATR-based dynamic width |
| `atr_multiplier` | number | 2.0 | 0.5-5.0 | ATR multiplier when range_mode=atr_adaptive |
| `atr_period` | number | 14 | 5-50 | ATR lookback period in candles |
| `check_interval` | duration | 1h | 1m-24h | How often to check price drift |
| `max_rebalance_freq` | duration | 4h | 1h-48h | Minimum time between rebalances |
| `slippage_tolerance` | percent | 1% | 0.1%-5% | Max slippage for swap during rebalance |

## Decision Logic

```
EVERY check_interval:

1. FETCH pool_state → current_price
2. FETCH position → { lower_tick, upper_tick, liquidity, unclaimed_fees }
3. CALCULATE range_center = (upper_tick + lower_tick) / 2
4. CALCULATE drift = |current_price - range_center| / range_center

5. IF drift < drift_threshold:
     → HOLD (position is in range, no action needed)

6. IF drift >= drift_threshold:
     a. CHECK rebalance-cooldown → if too recent, HOLD + log "cooldown active"
     b. CHECK gas-check → if gas too high relative to position, HOLD + log
     c. CHECK il-guard → if IL above threshold, ALERT user instead of rebalance

     d. CALCULATE new_range:
        IF range_mode == "fixed":
          new_lower = current_price * (1 - range_width / 2)
          new_upper = current_price * (1 + range_width / 2)
        IF range_mode == "atr_adaptive":
          atr = FETCH market_indicators(ATR, atr_period)
          dynamic_width = atr * atr_multiplier / current_price
          new_lower = current_price * (1 - dynamic_width / 2)
          new_upper = current_price * (1 + dynamic_width / 2)

     e. EXECUTE rebalance:
        1. remove-liquidity(position_id) → receive token_a, token_b
        2. CALCULATE target ratio for new range
        3. IF current ratio != target ratio:
             swap(excess_token → deficit_token, slippage_tolerance)
        4. add-liquidity(pool, new_lower, new_upper, token_a, token_b)

     f. LOG rebalance event: old_range, new_range, fees_collected, swap_cost
```

## Exit Conditions

| Condition | Action |
|-----------|--------|
| IL exceeds `il-guard` threshold | Alert user, suggest exit |
| User requests exit | remove-liquidity → swap to desired token → done |
| Pool TVL drops below safety threshold | Alert user, suggest migration |
| Strategy manually stopped | Positions remain open, stop monitoring |

## Performance Metrics

Track and report to user:
- Total fees earned (cumulative)
- Total rebalance count
- Total swap costs from rebalances
- Net P&L (fees - swap costs - IL)
- Current APR (annualized from recent period)
- Time in range %

## Example Configurations

### Conservative (wide range, infrequent rebalance)
```
pool: SOL/USDC
drift_threshold: 8%
range_width: 20%
check_interval: 4h
max_rebalance_freq: 12h
```

### Moderate (default)
```
pool: SOL/USDC
drift_threshold: 5%
range_width: 10%
check_interval: 1h
max_rebalance_freq: 4h
```

### Aggressive (tight range, frequent rebalance)
```
pool: SOL/USDC
drift_threshold: 2%
range_width: 5%
range_mode: atr_adaptive
atr_multiplier: 1.5
check_interval: 15m
max_rebalance_freq: 1h
```
