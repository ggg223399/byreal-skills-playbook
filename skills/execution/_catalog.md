# Execution Module Catalog

Atomic execution actions that strategies compose into flows. Each module describes the operation, required inputs, and expected outputs.

| Module | Description | Chain | File |
|--------|-------------|-------|------|
| **add-liquidity** | Deploy a new LP position with specified range and amount | Solana (Byreal) | `add-liquidity.md` |
| **remove-liquidity** | Withdraw an existing LP position (full or partial) | Solana (Byreal) | `remove-liquidity.md` |
| **collect-fees** | Claim unclaimed fees from an LP position | Solana (Byreal) | `collect-fees.md` |
| **rebalance** | Atomic remove + swap (if needed) + re-deploy at new range | Solana (Byreal) | `rebalance.md` |
| **swap** | Token swap via Byreal DEX router | Solana (Byreal) | `swap.md` |

## Composition

Strategies chain execution modules into flows. Example:

```
Auto Range Rebalance:
  remove-liquidity → swap (balance tokens) → add-liquidity

Auto Compound:
  collect-fees → swap (to pool tokens) → add-liquidity

IL Monitor + Auto Exit:
  remove-liquidity → swap (to stable) → done
```

## Safety

- All execution modules perform pre-flight checks before submitting transactions
- Slippage tolerance is configurable (default: 1%)
- Gas estimation is checked against position value (see `risk/gas-check.md`)
- Failed transactions are retried once with increased priority fee, then reported to user
