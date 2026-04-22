---
name: quant-backtest-system
description: Quant backtesting web system - FastAPI + ECharts. Fully built and tested at /opt/hermes/quant-system/
trigger: User asks about quant backtesting, trading signals, strategy testing, or K-line charts
---

# Quant Backtest Web System (小宇量化风格)

## Status: ✅ Built and Working

**Location**: `/opt/hermes/quant-system/`
**Access**: `http://localhost:3001` (port 3001, not 3000 — 3000 was in use)
**Venv**: `/opt/hermes/ai-hedge-fund/.venv/`

## Start Command
```bash
cd /opt/hermes/quant-system/backend
/opt/hermes/ai-hedge-fund/.venv/bin/python -m uvicorn main:app --host 0.0.0.0 --port 3001
```

## Architecture
```
/opt/hermes/quant-system/
├── backend/
│   ├── main.py              # FastAPI entry (uses HTMLResponse, NOT Jinja2 TemplateResponse)
│   ├── routers/             # market.py, backtest.py, strategy.py, scan.py
│   ├── services/            # data_service.py, backtest_service.py, ai_service.py
│   └── models/schemas.py    # Pydantic models
├── frontend/
│   ├── templates/           # index.html, strategy.html
│   └── static/css/ + js/    # ECharts + Bootstrap 5 (all CDN)
├── data/                    # Parquet cache (auto-generated test data fallback)
└── start.sh
```

## API Endpoints
| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | K-line backtest UI |
| GET | `/strategy` | AI strategy generation UI |
| GET | `/api/kline?code=600519&period=daily&start=20250101&end=20260417&indicators=MA5,MA20,MACD` | K-line + indicators |
| POST | `/api/backtest` | Run backtest (JSON: code, strategy_name, start, end, cash) |
| GET | `/api/strategies` | List built-in strategies |
| POST | `/api/strategy/generate` | Text description → Python code |
| POST | `/api/strategy/explain` | Code → Chinese description |
| GET | `/api/scan?strategy=ma_cross&pool=600519,300750` | Strategy radar scan |

## 6 Built-in Strategies
ma_cross, rsi, macd, boll, kdj, turtle

## Critical Gotchas (learned from debugging)

### 1. Indicator Name Parsing — MACD vs MA prefix collision
`ind.startswith("MA")` matches BOTH "MA5" AND "macd" (uppercased). Must check exact matches (MACD, RSI, KDJ, BOLL, ATR) BEFORE prefix matches (EMA, MA). Also guard: `int(ind[2:]) if ind[2:].isdigit() else 20`.

### 2. Network Timeout — AkShare hangs forever
AkShare HTTP calls block indefinitely when network is down. Must check connectivity first with `socket.create_connection(("8.8.8.8", 53), timeout=3)`. If network unreachable, fall back to `generate_test_data()` (random walk price simulation).

### 3. Template Rendering — Jinja2 TemplateResponse fails
`TemplateResponse("index.html", {"request": request})` threw 500 errors. Solution: use `HTMLResponse(content=path.read_text())` instead. Simpler and works reliably.

### 4. Port Conflict — 3000 in use
Port 3000 was already occupied. Use 3001. Check with `ss -tlnp | grep <port>` or just try a different port.

### 5. pip Not in PATH — venv has no pip binary
The ai-hedge-fund venv has Python but no `pip` script. Install pip via: `python3 /tmp/get-pip.py --target=<venv>/lib/pythonX.Y/site-packages/`. Then use `python -m pip install ...`.

### 6. Pydantic v2 — response_model needs .tolist() not numpy arrays
When returning indicator data through Pydantic models, must convert numpy arrays to Python lists with `.tolist()`. Pydantic v2 is strict about types.

### 7. A-share Trading Rules
- 100 shares minimum, 100-share increments
- Commission: 万3 (0.03%), min 5 yuan
- Stamp tax: 千1 (0.1%), sell only
- T+1 settlement

## Technical Indicators (Pure Python, no ta-lib)
All implemented in `data_service.py`: MA, EMA, MACD, RSI, KDJ, BOLL, ATR

## Network Fallback
When AkShare can't connect, `generate_test_data()` creates deterministic random-walk price data seeded by stock code hash. Ensures system always works for demo/testing.

## Dependencies (already installed in ai-hedge-fund venv)
fastapi, uvicorn, pandas, numpy, akshare, jinja2, python-multipart, aiofiles
