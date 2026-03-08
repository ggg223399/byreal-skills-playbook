# Data Feed: Market Indicators

> Price candles and technical indicators for strategy decision-making.

## Source

Byreal Kline API

## Fetch

```
GET /api/kline/{token_pair}?interval={interval}&limit={limit}
GET /api/indicators/{token_pair}?indicators={list}&interval={interval}
```

### Intervals

`1m`, `5m`, `15m`, `1h`, `4h`, `1d`, `1w`

## Output Schema

### Candles (OHLCV)

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | number | Candle open time |
| `open` | number | Opening price |
| `high` | number | Highest price |
| `low` | number | Lowest price |
| `close` | number | Closing price |
| `volume` | number | Trading volume (USDC) |

### Indicators

Request specific indicators. Each returns its current value and recent history.

| Indicator | Params | Output Fields |
|-----------|--------|---------------|
| **ATR** (Average True Range) | `period` (default: 14) | `atr`, `atr_percent` (ATR / price) |
| **Bollinger Bands** | `period` (20), `std_dev` (2) | `upper`, `middle`, `lower`, `bandwidth` |
| **RSI** | `period` (14) | `rsi` (0-100) |
| **MA** (Moving Average) | `period`, `type` (SMA/EMA) | `value` |
| **MACD** | `fast` (12), `slow` (26), `signal` (9) | `macd`, `signal`, `histogram` |
| **Volume Profile** | `period` (24h) | `poc` (point of control), `value_area_high`, `value_area_low` |

## Usage in Strategies

| Strategy | Indicators Used | Purpose |
|----------|----------------|---------|
| **#1 Auto Range Rebalance** | ATR | Dynamic range width (atr_adaptive mode) |
| **#2 Volatility-Adaptive** | ATR, Bollinger bandwidth | Detect volatility regime changes |
| **#3 Narrow Range Sniper** | ATR, Bollinger | Identify compression before expansion |
| **#6 Mean-Reversion Range** | Bollinger, MA | Detect mean-deviation signals |
| **#7 Breakout Range** | Bollinger, Volume | Confirm breakout validity |

## Refresh Frequency

| Indicator | Recommended Interval |
|-----------|---------------------|
| ATR (1h candles) | Every 15-60 min |
| Bollinger (1h) | Every 15-60 min |
| RSI (1h) | Every 15-60 min |
| MA (daily) | Every 1-4 h |
