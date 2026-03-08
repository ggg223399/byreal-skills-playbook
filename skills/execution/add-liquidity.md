# Execution: Add Liquidity

> Deploy a new LP position or add liquidity to an existing position on Byreal CLMM.

## Operation

Create a concentrated liquidity position with a specified price range and token amounts.

## Inputs

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `pool` | string | yes | Pool address or token pair |
| `lower_price` | number | yes | Lower bound of price range |
| `upper_price` | number | yes | Upper bound of price range |
| `token_a_amount` | number | yes* | Amount of token A to deposit |
| `token_b_amount` | number | yes* | Amount of token B to deposit |
| `slippage_tolerance` | percent | no (default: 1%) | Max acceptable slippage |

*Either provide both amounts, or one amount and let the system calculate the other based on the range and current price.

## Pre-flight Checks

1. Wallet has sufficient balance of both tokens
2. Price range is valid (lower < current < upper for balanced entry)
3. Tick spacing is compatible with pool's tick spacing
4. Gas estimate is acceptable (see `risk/gas-check`)

## Output

| Field | Type | Description |
|-------|------|-------------|
| `position_address` | string | New position on-chain address |
| `tx_signature` | string | Transaction signature |
| `actual_token_a` | number | Actual token A deposited |
| `actual_token_b` | number | Actual token B deposited |
| `liquidity` | number | Liquidity amount created |

## Error Handling

| Error | Recovery |
|-------|----------|
| Insufficient balance | Abort, notify user with required amounts |
| Slippage exceeded | Retry with higher tolerance or wait for calmer market |
| Transaction failed | Retry once with higher priority fee, then abort |
