# Risk Guard Catalog

Pre-flight and runtime risk checks. Strategies declare required risk guards in their `requires.risk` section. Guards run before every execution action and can BLOCK, WARN, or PASS.

| Guard | Description | Action on Trigger | File |
|-------|-------------|-------------------|------|
| **il-guard** | Monitor impermanent loss against threshold | BLOCK exit or ALERT user | `il-guard.md` |
| **gas-check** | Ensure transaction cost is reasonable relative to position value | BLOCK if gas > threshold % | `gas-check.md` |
| **max-drawdown** | Portfolio-level circuit breaker on cumulative loss | BLOCK all actions, alert user | `max-drawdown.md` |
| **rebalance-cooldown** | Prevent excessive rebalancing (fee churn) | BLOCK rebalance if too recent | `rebalance-cooldown.md` |
| **position-size** | Check that position size is within acceptable bounds | WARN if oversized | `position-size.md` |

## Guard Response

```
PASS  → proceed with execution
WARN  → log warning, proceed (notify user async)
BLOCK → halt execution, notify user immediately
```

## Override

User can override WARN-level guards. BLOCK-level guards require explicit user confirmation to override. The `max-drawdown` guard (kill switch) cannot be overridden without re-authentication.
