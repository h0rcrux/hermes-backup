---
name: akshare-a-stock
description: Integrate Akshare for A-share (Chinese stock market) data in trading/analytics systems. Covers fallback strategies when Docker/Java/Maven are unavailable, creating Node.js mock backends, and multi-source data architecture.
tags: [akshare, a-stock, china, trading, quant, fallback, mock-backend]
---

# Akshare A-Stock Data Integration

## When to Use
- Building quantitative trading systems for Chinese A-share market
- Need free, no-API-key stock data sources
- Docker/Java/Maven infrastructure unavailable (fallback approach)
- Replacing FMP (US-focused) with A-share data source

## Akshare Capabilities
- **Library**: Python, `pip install akshare`
- **Version**: 1.18.55+ (as of 2026)
- **Data Types**: Historical K-lines, real-time quotes, financial reports, fund flows
- **Limitations**: 15-minute delay on real-time data, request rate limits

## Key API Functions
```python
import akshare as ak

# Historical daily K-line
df = ak.stock_zh_a_hist(symbol="000001", period="daily", 
                         start_date="20240101", end_date="20240131", adjust="qfq")

# Index data (more stable)
df = ak.stock_zh_index_daily(symbol="sh000001")  # Shanghai Composite

# Stock list
df = ak.stock_info_a_code_name()  # Returns {code, name}

# Real-time quotes (may have connection issues)
df = ak.stock_zh_a_spot_em()
```

## Known Issues & Solutions

### Connection Errors
Some akshare endpoints have unstable connections. Solutions:
1. Use `stock_zh_index_daily` instead of `stock_zh_a_hist` (more stable)
2. Implement retry logic with exponential backoff
3. Create mock data fallback for demo/testing

### Stock Code Formats
- Shenzhen Main: `000001-000999`
- Shenzhen SME: `002001-002999`
- Shenzhen ChiNext: `300001-300999`
- Shanghai Main: `600000-600999`
- Shanghai STAR: `688001-688999`

## Fallback: Node.js Mock Backend

When Docker/Java/Maven unavailable, create Express.js mock:

```javascript
// mock-backend/server.js
const express = require('express');
const cors = require('cors');
const app = express();

app.use(cors());

// Mock K-line data with indicators
app.get('/api/kline', (req, res) => {
  res.json({
    klines: [/* OHLCV data */],
    indicators: {
      ma: { ma5List: [], ma10List: [], ma20List: [] },
      macd: { difList: [], deaList: [], macdList: [] },
      rsi: { rsi6List: [], rsi12List: [], rsi24List: [] },
      boll: { upperList: [], middleList: [], lowerList: [] }
    }
  });
});

app.listen(8181);
```

## Integration Pattern: Multi-Source Architecture

```
Frontend (React)
    ↓
API Gateway (web-service: 8181)
    ↓
┌─────────────────────────────┐
│  FMP Service (US stocks)    │ 8182
│  Akshare Service (A-stocks) │ 8183
│  Mock Service (fallback)    │ 8184
└─────────────────────────────┘
```

Frontend selects data source based on market:
```typescript
if (market === 'cn') {
  data = await fetchAkshareKline(symbol, period, startDate, endDate);
} else {
  data = await fetchKlineWithIndicators(symbol, market, period, startDate, endDate);
}
```

## Service Template (Node.js)

```javascript
// akshare-service/server.js
const express = require('express');
const app = express();

// Endpoints
app.get('/api/akshare/search', searchStocks);
app.get('/api/akshare/kline', getKline);
app.get('/api/akshare/stocks', getStockList);

// Technical indicators calculation (client-side)
function calculateIndicators(klines) {
  // MA, MACD, RSI, BOLL calculations
  return { ma, macd, rsi, boll };
}
```

## Launch Scripts Pattern

```bash
#!/bin/bash
# start-system.sh
node mock-backend/server.js &      # Port 8181
node akshare-service/server.js &   # Port 8182
cd web-app && npm run dev &        # Port 3000
wait
```

## Pitfalls

1. **Don't rely on real-time data**: 15-minute delay is standard for free sources
2. **Rate limiting**: Space requests 1-2 seconds apart
3. **Connection instability**: Always implement retry/fallback
4. **Data format mismatch**: Normalize akshare output to your internal format
5. **Missing indicators**: Calculate MA/MACD/RSI/BOLL client-side if not provided

## Comparison: Data Sources

| Source | Free | Real-time | A-Share | Stability |
|--------|------|-----------|---------|-----------|
| Akshare | ✅ | ❌ (15min) | ✅ | Medium |
| FMP | ✅ | ✅ | ❌ | High |
| Tushare | ✅ | ❌ | ✅ | High |
| EastMoney | ✅ | ✅ | ✅ | High |

## Environment Setup

```bash
pip install akshare
npm install express cors axios
```

## Multi-Agent Parallel Development

When building complex trading systems, delegate modules to parallel subagents:

```
Batch 1: Foundation (trading-common, database scripts)
Batch 2: Core services (market-data, indicator, strategy) - parallel
Batch 3: Integration services (backtest, web-service) - parallel
Batch 4: Frontend (React app)
```

Each subagent gets isolated context with file/terminal toolsets. Dependencies are resolved by batch ordering.

## Project Structure (Quant Trading System)

```
quant-trading-system/
├── trading-common/        # Shared DTOs
├── market-data-service/   # Data fetching (port 8182)
├── indicator-service/     # Technical indicators (port 8183)
├── strategy-core/         # Trading strategies
├── backtest-service/      # Backtesting engine (port 8185)
├── web-service/           # BFF layer (port 8181)
├── web-app/               # React frontend (port 3000)
├── mock-backend/          # Fallback when Java unavailable
├── akshare-service/       # A-stock data source (port 8182)
├── scripts/               # start-system.sh, stop-system.sh
└── docs/                  # QUICK_START.md, guides
```

## References
- [Akshare Docs](https://akshare.akfamily.xyz/)
- [Akshare GitHub](https://github.com/akfamily/akshare)
- [Project: /opt/data/quant-trading-system/]
