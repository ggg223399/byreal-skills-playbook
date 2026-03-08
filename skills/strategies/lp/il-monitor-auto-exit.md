# Strategy #18: IL Monitor + Auto Exit

> Continuously monitor impermanent loss and automatically exit the LP position when IL exceeds a threshold.

## Overview

| Attribute | Value |
|-----------|-------|
| Category | LP — Lifecycle Management |
| Risk | Conservative (C) |
| Complexity | * Single-step |
| Market | High Volatility (V) |
| Cycle | H |
| Min Capital | 500 USDC |

## Requires

```yaml
feeds:
  - pool-state       # current price
  - position         # position value, IL calculation, unclaimed fees

execution:
  - remove-liquidity
  - collect-fees
  - swap

risk:
  - gas-check
```

## Parameters

| Param | Type | Default | Constraints | Description |
|-------|------|---------|-------------|-------------|
| `pool` | string | — | required | Pool address or token pair |
| `position_id` | string | — | required (or auto-detect) | Position to monitor |
| `il_warn_threshold` | percent | 3% | 0.5%-20% | IL level that triggers a warning to user |
| `il_exit_threshold` | percent | 5% | 1%-30% | IL level that triggers automatic exit |
| `il_calc_mode` | enum | vs_hold | vs_hold / vs_initial_usd | Calculate IL vs holding tokens or vs initial USD value |
| `net_loss_mode` | bool | true | — | If true, offset IL with earned fees (exit only if net loss > threshold) |
| `check_interval` | duration | 30m | 1m-4h | How often to check IL |
| `exit_to` | enum | usdc | usdc / token_a / token_b / balanced | What to swap to after exit |
| `alert_channel` | enum | log | log / notify | How to deliver warnings |

## Decision Logic

```
EVERY check_interval:

1. FETCH position → {
     token_a_amount, token_b_amount,
     initial_token_a, initial_token_b,
     initial_value_usd,
     unclaimed_fee_a, unclaimed_fee_b
   }
2. FETCH pool_state → current_price

3. CALCULATE il:
     IF il_calc_mode == "vs_hold":
       hold_value = initial_token_a * current_price_a + initial_token_b * current_price_b
       position_value = token_a_amount * current_price_a + token_b_amount * current_price_b
       il = (position_value - hold_value) / hold_value    # negative = loss
     IF il_calc_mode == "vs_initial_usd":
       il = (current_position_value_usd - initial_value_usd) / initial_value_usd

4. IF net_loss_mode:
     total_fees_usd = (unclaimed_fee_a + collected_fee_a) * price_a + ...
     net_il = il + (total_fees_usd / initial_value_usd)
     → use net_il for threshold comparisons
   ELSE:
     → use raw il

5. IF |il| < il_warn_threshold:
     → HOLD (position healthy)

6. IF |il| >= il_warn_threshold AND |il| < il_exit_threshold:
     → WARN user: "IL at {il}%, approaching exit threshold {il_exit_threshold}%"
     → increase check frequency to check_interval / 2 (temporary)

7. IF |il| >= il_exit_threshold:
     → EXECUTE exit:
       a. collect-fees(position_id) → receive unclaimed fees
       b. remove-liquidity(position_id) → receive token_a, token_b
       c. IF exit_to == "usdc":
            swap(token_a → USDC)
            swap(token_b → USDC)  # skip if already USDC
          IF exit_to == "token_a":
            swap(token_b → token_a)
          IF exit_to == "token_b":
            swap(token_a → token_b)
          IF exit_to == "balanced":
            → keep as-is (both tokens)

     → NOTIFY user:
       "Position exited. IL was {il}%.
        Fees earned: {total_fees} USDC
        Net P&L: {net_pnl} USDC
        Funds returned as: {exit_to}"

     → STOP strategy (position no longer exists)
```

## Alert Escalation

```
IL < warn     →  Green  — check at normal interval
IL >= warn    →  Yellow — warn user, increase check frequency
IL >= exit    →  Red    — auto exit, notify user
```

## Exit Conditions

| Condition | Action |
|-----------|--------|
| IL >= exit threshold | Auto exit (core behavior) |
| Pool TVL drops > 50% in 1h | Emergency alert + suggest exit |
| User requests exit | Execute exit flow |
| User overrides warn | Acknowledge and continue monitoring |

## Performance Metrics

Track and report:
- Current IL (real-time)
- IL history (chart over time)
- Fees earned vs IL comparison
- Net P&L at time of exit
- Time in position before exit
- Peak IL during position lifetime

## Interaction with Other Strategies

IL Monitor works as a **safety layer** on top of other LP strategies:

| Primary Strategy | Combo Behavior |
|-----------------|----------------|
| #1 Auto Range Rebalance | IL Monitor acts as emergency brake; rebalance handles normal drift |
| #9 Auto Compound | IL Monitor pauses compound at warn level; exits at exit level |
| #4 Multi-Range Split | Monitor IL per sub-position, can exit individual ranges |

When combined, IL Monitor's `il_exit_threshold` should be **higher** than Auto Range Rebalance's drift threshold — rebalance is the first line of defense, IL exit is the last resort.

## Example Configurations

### Tight Protection (stablecoin pairs)
```
pool: USDC/USDT
il_warn_threshold: 0.5%
il_exit_threshold: 1%
net_loss_mode: true
check_interval: 15m
exit_to: usdc
```

### Standard (volatile pairs)
```
pool: SOL/USDC
il_warn_threshold: 3%
il_exit_threshold: 5%
net_loss_mode: true
check_interval: 30m
exit_to: usdc
```

### Loose (high-yield degen pools)
```
pool: BONK/SOL
il_warn_threshold: 10%
il_exit_threshold: 20%
net_loss_mode: true
check_interval: 1h
exit_to: balanced
```
