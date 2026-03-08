# Execution: Collect Fees

> Claim unclaimed trading fees from an LP position without removing liquidity.

## Operation

Collect accrued fees from a position. The position remains active and continues earning.

## Inputs

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `position_id` | string | yes | Position address |

## Pre-flight Checks

1. Position exists and is owned by agent wallet
2. Unclaimed fees > 0 (skip if nothing to collect)
3. Gas cost < fee value (see `risk/gas-check`)

## Output

| Field | Type | Description |
|-------|------|-------------|
| `tx_signature` | string | Transaction signature |
| `fee_a_collected` | number | Fee token A collected |
| `fee_b_collected` | number | Fee token B collected |
| `total_usd` | number | Total fee value in USDC |

## Notes

- Fees are sent to the agent wallet
- Position liquidity is unchanged
- No slippage concern (fees are exact amounts)
