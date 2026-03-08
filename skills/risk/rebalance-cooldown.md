# Risk Guard: Rebalance Cooldown

> Prevent excessive rebalancing that erodes returns through swap fees and gas.

## Trigger

Runs before any rebalance execution action.

## Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `min_interval` | duration | 4h | Minimum time between rebalances for the same position |
| `max_daily` | number | 3 | Maximum rebalances per 24h per position |

## Logic

```
1. FETCH last_rebalance_time for this position
2. CALCULATE time_since = now - last_rebalance_time

3. IF time_since < min_interval:
     → BLOCK "Cooldown active. Next rebalance allowed in {min_interval - time_since}."

4. FETCH rebalance_count_24h for this position
5. IF rebalance_count_24h >= max_daily:
     → BLOCK "Daily rebalance limit reached ({max_daily}/{max_daily}). Resets in {time_until_reset}."

6. ELSE:
     → PASS
```

## Cost Tracking

Each rebalance logs its cost (swap fee + gas). If cumulative daily rebalance costs > 50% of daily fee earnings, emit:
→ WARN "Rebalance costs ({cost}) consuming {ratio}% of daily fees ({fees}). Consider widening range."

## Notes

- Prevents "churn" in volatile markets where price oscillates around range boundaries
- Strategy #1 (Auto Range Rebalance) uses `max_rebalance_freq` param which feeds into this guard
- Users can override BLOCK with explicit confirmation, but the cost warning remains
