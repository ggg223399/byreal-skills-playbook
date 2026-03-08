# Data Feed: Position

> Current state of a user's LP position(s) on Byreal CLMM.

## Source

Byreal DEX API

## Fetch

```
GET /api/position/{position_address}
GET /api/positions/by-owner/{wallet_address}
GET /api/positions/by-pool/{pool_address}?owner={wallet_address}
```

## Output Schema

| Field | Type | Unit | Description |
|-------|------|------|-------------|
| `position_address` | string | — | On-chain position address |
| `pool_address` | string | — | Pool this position belongs to |
| `owner` | string | — | Wallet address of position owner |
| `lower_tick` | number | — | Lower bound tick index |
| `upper_tick` | number | — | Upper bound tick index |
| `lower_price` | number | token_b/a | Lower bound price |
| `upper_price` | number | token_b/a | Upper bound price |
| `liquidity` | number | — | Liquidity amount deposited |
| `token_a_amount` | number | token_a | Current token A in position |
| `token_b_amount` | number | token_b | Current token B in position |
| `unclaimed_fee_a` | number | token_a | Unclaimed fee in token A |
| `unclaimed_fee_b` | number | token_b | Unclaimed fee in token B |
| `in_range` | bool | — | Whether current price is within position range |
| `position_value_usd` | number | USDC | Total position value at current prices |
| `initial_deposit_usd` | number | USDC | Value at time of deposit |
| `il_percent` | number | percent | Impermanent loss vs holding |
| `fees_earned_usd` | number | USDC | Total fees earned (claimed + unclaimed) |
| `net_pnl_usd` | number | USDC | Position value + fees - initial deposit |
| `created_at` | timestamp | — | When position was opened |

## Derived Calculations

### Impermanent Loss

```
il = position_value / hold_value - 1

where:
  hold_value = initial_token_a * current_price_a + initial_token_b * current_price_b
  position_value = token_a_amount * current_price_a + token_b_amount * current_price_b
```

### Range Center & Width

```
range_center = (lower_price + upper_price) / 2
range_width = (upper_price - lower_price) / range_center
```

### Price Drift

```
drift = |current_price - range_center| / range_center
```

## Refresh Frequency

| Use Case | Recommended Interval |
|----------|---------------------|
| IL monitoring | 5-30 min |
| Fee tracking | 15-60 min |
| Rebalance check | 5-15 min |

## Usage in Strategies

- **Auto Range Rebalance**: `lower_tick`, `upper_tick`, `in_range` for drift detection
- **Auto Compound**: `unclaimed_fee_a/b` to check compound threshold
- **IL Monitor**: `il_percent`, `fees_earned_usd` for net IL calculation
