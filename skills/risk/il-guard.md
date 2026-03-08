# Risk Guard: IL Guard

> Monitor impermanent loss and enforce thresholds.

## Trigger

Runs before every strategy decision cycle and before execution actions that modify positions.

## Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `warn_threshold` | percent | 3% | IL level that triggers WARN |
| `block_threshold` | percent | 5% | IL level that triggers BLOCK |
| `net_mode` | bool | true | Offset IL with earned fees |

## Logic

```
1. FETCH position → il_percent, fees_earned_usd, initial_value_usd

2. IF net_mode:
     effective_il = |il_percent| - (fees_earned_usd / initial_value_usd)
   ELSE:
     effective_il = |il_percent|

3. IF effective_il < warn_threshold:
     → PASS

4. IF effective_il >= warn_threshold AND effective_il < block_threshold:
     → WARN "IL at {effective_il}% (raw: {il_percent}%, fees offset: {offset}%)"

5. IF effective_il >= block_threshold:
     → BLOCK "IL exceeded {block_threshold}%. Action blocked. User review required."
```

## Response

| Level | Effect |
|-------|--------|
| PASS | Strategy proceeds normally |
| WARN | Log warning, notify user async, strategy continues |
| BLOCK | Halt current action, notify user, require confirmation to proceed |

## Override

User can acknowledge WARN and continue. BLOCK requires explicit user confirmation with the current IL value displayed.
