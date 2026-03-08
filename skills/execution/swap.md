# Execution: Swap

> Execute a token swap via Byreal DEX router.

## Operation

Swap one token for another using the optimal route across Byreal pools.

## Inputs

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `token_in` | string | yes | Token mint or symbol to sell |
| `token_out` | string | yes | Token mint or symbol to buy |
| `amount_in` | number | yes* | Amount of token_in to swap |
| `amount_out` | number | yes* | Desired amount of token_out (alternative to amount_in) |
| `slippage_tolerance` | percent | no (default: 1%) | Max acceptable slippage |

*Provide either `amount_in` (exact input) or `amount_out` (exact output), not both.

## Pre-flight Checks

1. Wallet has sufficient balance of token_in
2. Route exists with acceptable price impact
3. Price impact < 3% (WARN if > 1%)
4. Gas estimate is acceptable

## Output

| Field | Type | Description |
|-------|------|-------------|
| `tx_signature` | string | Transaction signature |
| `amount_in_actual` | number | Actual tokens sold |
| `amount_out_actual` | number | Actual tokens received |
| `price` | number | Effective execution price |
| `price_impact` | percent | Price impact of the swap |
| `route` | string[] | Pools used in the route |

## Notes

- For large swaps, consider using TWAP (#27) or VWAP (#28) execution instead
- Price impact > 1% is logged as a warning
- Price impact > 5% requires user confirmation
