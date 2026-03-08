# Execution: Rebalance

> Atomic operation: remove liquidity from current range, swap to correct ratio, and re-deploy at a new range.

## Operation

Combines remove-liquidity + swap + add-liquidity into a single logical operation. This is the core action for range management strategies.

## Inputs

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `position_id` | string | yes | Current position to rebalance |
| `new_lower_price` | number | yes | New range lower bound |
| `new_upper_price` | number | yes | New range upper bound |
| `slippage_tolerance` | percent | no (default: 1%) | Max slippage for swap step |

## Flow

```
1. remove-liquidity(position_id, 100%)
   → receive token_a, token_b, fees

2. Calculate target token ratio for new range at current price
   → target_ratio = f(new_lower, new_upper, current_price)

3. IF current_ratio != target_ratio:
     swap(excess_token → deficit_token)
     → now balanced for new range

4. add-liquidity(pool, new_lower, new_upper, token_a, token_b)
   → new position deployed
```

## Pre-flight Checks

1. All remove-liquidity pre-flight checks
2. New range is valid (lower < upper, compatible tick spacing)
3. Combined gas estimate (3 txs) is acceptable vs position value
4. Rebalance cooldown has elapsed (see `risk/rebalance-cooldown`)

## Output

| Field | Type | Description |
|-------|------|-------------|
| `old_position_id` | string | Closed position |
| `new_position_id` | string | New position address |
| `old_range` | [number, number] | Previous [lower, upper] |
| `new_range` | [number, number] | New [lower, upper] |
| `swap_cost` | number | Cost of ratio rebalancing swap (USDC) |
| `fees_collected` | number | Fees collected during removal (USDC) |
| `total_gas` | number | Total gas for all transactions (USDC) |
| `net_cost` | number | swap_cost + gas - fees_collected |

## Error Handling

| Error | Recovery |
|-------|----------|
| Remove succeeds but swap fails | Tokens are in wallet; retry swap or let user decide |
| Remove + swap succeed but add fails | Tokens are in wallet; retry add or let user decide |
| Partial failure | Never leave funds stuck — worst case tokens sit in wallet |

## Cost Awareness

Rebalancing has real costs:
- Swap fee (pool fee rate, typically 0.3%)
- Swap slippage
- Gas (3 transactions)

Log each rebalance cost. If cumulative rebalance costs exceed fee earnings, WARN the user — the strategy may need wider ranges or less frequent checks.
