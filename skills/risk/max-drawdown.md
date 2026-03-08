# Risk Guard: Max Drawdown

> Portfolio-level circuit breaker. Halts all strategy activity when cumulative losses exceed threshold.

## Trigger

Runs on every strategy check cycle. Evaluates across all active positions managed by the agent.

## Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `max_drawdown` | percent | 10% | Total portfolio drawdown that triggers kill switch |
| `warn_drawdown` | percent | 7% | Drawdown level that triggers warning |
| `lookback` | duration | 7d | Window for drawdown calculation |
| `scope` | enum | all | `all` = total portfolio / `strategy` = per-strategy |

## Logic

```
1. CALCULATE portfolio_value = sum of all position values + wallet balance
2. CALCULATE peak_value = max portfolio_value within lookback period
3. CALCULATE drawdown = (peak_value - portfolio_value) / peak_value

4. IF drawdown < warn_drawdown:
     → PASS

5. IF drawdown >= warn_drawdown AND drawdown < max_drawdown:
     → WARN "Portfolio drawdown at {drawdown}%. Peak: {peak_value}, Current: {portfolio_value}"

6. IF drawdown >= max_drawdown:
     → BLOCK ALL STRATEGIES
     → NOTIFY user: "KILL SWITCH ACTIVATED. Drawdown {drawdown}% exceeds {max_drawdown}% limit.
        All strategies paused. Review positions and confirm to resume."
```

## Kill Switch Behavior

When triggered:
1. All running strategy loops are paused
2. No new execution actions are allowed
3. Existing positions are **not** automatically closed (user decides)
4. User must explicitly resume after reviewing

## Resume

User must confirm with current portfolio state displayed. Optionally:
- Resume all strategies
- Resume selectively
- Exit all positions
- Adjust drawdown threshold

## Notes

- This is the last line of defense — individual strategy risk guards should catch most issues earlier
- Links to Strategy #117 (Kill Switch) in the playbook
- Cannot be overridden without user re-authentication
