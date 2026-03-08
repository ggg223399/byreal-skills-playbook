# Data Feed: Pool State

> Real-time state of a Byreal CLMM pool.

## Source

Byreal DEX API

## Fetch

```
GET /api/pool/{pool_address}
GET /api/pool/by-pair/{token_a}/{token_b}
```

## Output Schema

| Field | Type | Unit | Description |
|-------|------|------|-------------|
| `pool_address` | string | — | On-chain pool address |
| `token_a` | object | — | `{ mint, symbol, decimals }` |
| `token_b` | object | — | `{ mint, symbol, decimals }` |
| `current_price` | number | token_b per token_a | Current pool price |
| `current_tick` | number | — | Current active tick index |
| `tick_spacing` | number | — | Minimum tick increment for this pool |
| `fee_rate` | number | bps | Pool fee rate (e.g., 30 = 0.30%) |
| `tvl` | number | USDC | Total value locked |
| `volume_24h` | number | USDC | 24-hour trading volume |
| `fee_24h` | number | USDC | 24-hour fees generated |
| `apr_24h` | number | percent | Annualized fee APR based on 24h |
| `liquidity` | number | — | Total active liquidity |
| `token_a_reserve` | number | token_a | Pool token A balance |
| `token_b_reserve` | number | token_b | Pool token B balance |

## Refresh Frequency

| Use Case | Recommended Interval |
|----------|---------------------|
| Range monitoring | 1-5 min |
| Yield tracking | 15-60 min |
| Pool comparison | 1-6 h |

## Usage in Strategies

- **Auto Range Rebalance**: `current_price` to check drift from range center
- **Auto Compound**: `current_price` to calculate swap ratios for fee reinvestment
- **IL Monitor**: `current_price` to compute impermanent loss
- **Fee Tier Optimizer**: `fee_rate`, `apr_24h` to compare across tiers
- **Pool Rotation**: `apr_24h`, `tvl` to rank pools
