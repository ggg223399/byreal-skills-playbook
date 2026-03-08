# Execution: Remove Liquidity

> Withdraw liquidity from an existing LP position on Byreal CLMM. Can be full or partial withdrawal.

## Operation

Remove all or part of the liquidity from a position. Unclaimed fees are collected automatically during removal.

## Inputs

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `position_id` | string | yes | Position address to withdraw from |
| `percentage` | percent | no (default: 100%) | Portion of liquidity to remove |
| `slippage_tolerance` | percent | no (default: 1%) | Max acceptable slippage |
| `close_position` | bool | no (default: true if 100%) | Close the position account after full withdrawal |

## Pre-flight Checks

1. Position exists and is owned by the agent wallet
2. Position has liquidity > 0
3. Gas estimate is acceptable

## Output

| Field | Type | Description |
|-------|------|-------------|
| `tx_signature` | string | Transaction signature |
| `token_a_received` | number | Token A withdrawn |
| `token_b_received` | number | Token B withdrawn |
| `fee_a_collected` | number | Unclaimed fee A collected |
| `fee_b_collected` | number | Unclaimed fee B collected |
| `remaining_liquidity` | number | Liquidity remaining (if partial) |

## Error Handling

| Error | Recovery |
|-------|----------|
| Position not found | Check position_id, list user positions |
| Transaction failed | Retry once, then abort and notify user |
