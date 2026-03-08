# Risk Guard: Gas Check

> Ensure transaction costs are reasonable relative to the value being transacted.

## Trigger

Runs before every execution action (swap, add/remove liquidity, rebalance).

## Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `max_gas_ratio` | percent | 2% | Max gas cost as % of transaction value |
| `max_gas_absolute` | USDC | 5 | Absolute max gas in USDC (fallback for small positions) |

## Logic

```
1. ESTIMATE gas cost for the pending transaction(s)
2. CALCULATE transaction_value (position value or swap amount)
3. CALCULATE gas_ratio = gas_cost / transaction_value

4. IF gas_cost < max_gas_absolute AND gas_ratio < max_gas_ratio:
     → PASS

5. IF gas_ratio >= max_gas_ratio:
     → BLOCK "Gas ({gas_cost} USDC) is {gas_ratio}% of transaction value. Threshold: {max_gas_ratio}%."

6. IF gas_cost >= max_gas_absolute:
     → WARN "Gas unusually high: {gas_cost} USDC. Network may be congested."
```

## Notes

- On Solana, gas is typically very low (< $0.01), so this guard rarely triggers
- More relevant for multi-transaction operations (rebalance = 3 txs)
- During network congestion, priority fees can spike — this guard protects against that
