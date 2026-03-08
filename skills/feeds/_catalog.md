# Data Feed Catalog

Available data feeds that strategies can consume. Each feed module describes what data it provides, its API source, and the output schema.

| Feed | Description | Source | File |
|------|-------------|--------|------|
| **pool-state** | Pool price, TVL, volume, fee tier, tick spacing, liquidity distribution | Byreal DEX API | `pool-state.md` |
| **position** | Current LP positions — range, liquidity amount, unclaimed fees, IL calculation | Byreal DEX API | `position.md` |
| **market-indicators** | Price candles (OHLCV), technical indicators (ATR, Bollinger, RSI, MA, MACD) | Byreal Kline API | `market-indicators.md` |
| **incentive** | Pool incentive programs — reward token, APR, claim status, remaining duration | Byreal Incentive API | `incentive.md` |
| **on-chain-events** | Real-time swap events, large transactions, pool creation, TVL changes | Solana RPC / WebSocket | `on-chain-events.md` |
| **funding-rate** | Hyperliquid funding rates — current, predicted, historical | Hyperliquid API | `funding-rate.md` |

## Usage

Strategies declare required feeds in their `requires.feeds` section. The agent loads each feed module to understand:
1. What data is available
2. How to fetch it (API endpoints, parameters)
3. Output schema (fields, types, units)
4. Refresh frequency recommendations
