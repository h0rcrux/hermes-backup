---
name: ai-hedge-fund
description: Run the AI Hedge Fund project (virattt/ai-hedge-fund) to analyze stocks with 19+ AI analyst agents. Use when the user wants stock analysis, investment opinions, or trading signals.
tags: [stocks, finance, hedge-fund, analysis, trading]
---

# AI Hedge Fund — Stock Analysis with AI Agents

Located at `/opt/hermes/ai-hedge-fund`. Configured with NVIDIA NIM (minimaxai/minimax-m2.7) via `.env`.

## Hybrid Data Strategy: AkShare + QVeris (v2, IMPLEMENTED 2026-04-21)

**AkShare (free) → QVeris (paid fallback) — saves ~80% credits per stock.**

### File: `src/tools/akshare_provider.py` (CREATED)
Free data layer using akshare (Python library, no API key needed). Covers ~80% of what analysts need.
Functions: `get_historical_prices_akshare()`, `get_income_statement_akshare()`, `get_balance_sheet_akshare()`, `get_cash_flow_statement_akshare()`, `get_financial_metrics_akshare()`, `get_research_reports_akshare()`.

### Implementation Detail
`qveris_api.py` modified: `get_financial_metrics_qveris()` now tries akshare first, merges results into metrics_kwargs, only calls QVeris for PE/PB/PS/EV ratios (1 call). `search_line_items_qveris()` similarly tries akshare first for income/balance/cashflow rows. `prefetch_all_data()` uses akshare for 3 statement types, only QVeris for valuation + reports.

### AkShare Data Quality
- Income statement from Sina Finance has 90+ rows (all years), filter by `报告日.endswith("1231")` for annual
- Balance sheet includes: cash, short/long-term debt, goodwill, intangibles, equity — more granular than QVeris
- Cash flow: OCF + CapEx available (QVeris often returns N/A for these)
- Missing: D&A (estimate as 70% of CapEx), ROIC (compute from EBIT/total_capital)
- Real-time quotes (`stock_zh_a_spot_em()`) can timeout — not critical

### Data Flow (v2)
```
For A-share stocks:
  1. AkShare fetches: prices, income/balance/cashflow statements, financial metrics, ROIC, FCF
  2. QVeris fetches ONLY: PE/PB/PS/EV ratios, market cap, research reports (2 calls)
  3. If AkShare fails → full QVeris fallback (10-16 calls, old behavior)

For HK stocks:
  Still uses full QVeris (akshare doesn't support HK)
```

### Credit Cost Comparison
| Scenario | Before (v1) | After (v2) |
|----------|-------------|------------|
| Single A-share analysis | 30-60 credits | **5-10 credits** |
| 19 analysts on 1 stock | ~60 credits | **~10 credits** |
| Prefetch for 1 stock | 9-16 QVeris calls | **2 QVeris calls** |

### AkShare Coverage (what it provides free)
- ✅ OHLCV historical prices
- ✅ Income statement (revenue, COGS, op profit, net profit, R&D, interest expense)
- ✅ Balance sheet (cash, short/long-term debt, total assets/liabilities, equity, goodwill, intangibles)
- ✅ Cash flow statement (OCF, CapEx, dividends paid)
- ✅ Financial metrics (ROE, margins, ratios, YoY growth)
- ✅ Computed fields (ROIC, working capital, share count from EPS, D&A estimation)

### QVeris-only (still needs credits)
- PE/PB/PS/EV/PEG ratios from `mcp_gildata.stockvalueanalysis.v1`
- Research reports from `caidazi.report.query.v1.43d808fc`
- HK stock data (akshare doesn't support)

### How It Works
Each function in `qveris_api.py` now tries akshare first:
```python
# In get_financial_metrics_qveris():
try:
    from src.tools.akshare_provider import get_financial_metrics_akshare
    ak_metrics = get_financial_metrics_akshare(ticker, end_date, period, limit)
    if ak_metrics:
        # Merge akshare data, then only call QVeris for valuation ratios
        metrics_kwargs.update({k: v for k, v in ak_metrics.items() if v is not None})
        # QVeris: only 1 call for PE/PB/PS instead of 10+ calls
except Exception:
    # Full QVeris fallback
```

### AkShare Gotchas
- `stock_zh_a_hist()` connection errors are transient (rate limiting) — retry works
- `stock_financial_report_sina()` returns 90+ rows (all years), filter by `报告日.endswith("1231")` for annual
- D&A not directly in income statement — estimated as 70% of CapEx from cash flow
- `stock_a_indicator_lg()` doesn't exist in current akshare version — use THS abstract instead
- Real-time quotes (`stock_zh_a_spot_em()`) can timeout — not critical, valuation API has market cap

## Running Non-Interactively (Hermes terminal)

The CLI's `questionary` prompts hang in non-TTY contexts. **Do NOT use CLI** — call `run_hedge_fund()` directly via Python:

```bash
cd /opt/hermes/ai-hedge-fund && source .venv/bin/activate && PYTHONPATH=/opt/hermes/ai-hedge-fund QVERIS_API_KEY=sk-... python -c "
from src.main import run_hedge_fund
import json
result = run_hedge_fund(
    tickers=['AAPL'],
    start_date='2026-01-01',
    end_date='2026-04-19',
    portfolio={'cash': 100000, 'margin_requirement': 0.0, 'margin_used': 0.0,
               'positions': {'AAPL': {'long': 0, 'short': 0, 'long_cost_basis': 0.0, 'short_cost_basis': 0.0, 'short_margin_used': 0.0}},
               'realized_gains': {'AAPL': {'long': 0.0, 'short': 0.0}}},
    show_reasoning=True,
    selected_analysts=['fundamentals_analyst', 'valuation_analyst', 'technical_analyst', 'news_sentiment_analyst'],
    model_name='minimaxai/minimax-m2.7',
    model_provider='OpenAI',
)
print(json.dumps(result, indent=2, ensure_ascii=False, default=str))
"
```

Key notes:
- `--analysts-all` flag does NOT exist in current code — must use direct Python call
- Available analysts: `aswath_damodaran`, `ben_graham`, `bill_ackman`, `cathie_wood`, `charlie_munger`, `fundamentals_analyst`, `growth_analyst`, `michael_burry`, `mohnish_pabrai`, `nassim_taleb`, `news_sentiment_analyst`, `peter_lynch`, `phil_fisher`, `rakesh_jhunjhunwala`, `sentiment_analyst`, `stanley_druckenmiller`, `technical_analyst`, `valuation_analyst`, `warren_buffett`
- Must set `PYTHONPATH=/opt/hermes/ai-hedge-fund` or imports fail
- Must set `QVERIS_API_KEY` env var for A-share/HK data
- Running 3 vs 19 analysts uses **nearly the same QVeris credits** (~16 API calls) because data is prefetched once and shared across all analysts. The bottleneck is QVeris balance, not analyst count — run all 19 if credits allow.

## Supported Tickers

| Market | Ticker Format | Data Source | Status |
|---|---|---|---|
| US | `AAPL`, `MSFT` | Financial Datasets API | ✅ Full |
| A-share | `300750.SZ`, `600519.SH` | QVeris (Gildata) | ✅ Valuation + Price |
| Hong Kong | `00700.HK` | QVeris (Gildata) | ⚠️ Price only — no financials |

US tickers auto-route to Financial Datasets API. CN/HK tickers auto-route to QVeris via `src/tools/qveris_api.py`.

### Running A-share/HK Analysis
```bash
# A股 — 宁德时代
python src/main.py --tickers 300750.SZ --model minimaxai/minimax-m2.7 --analysts-all --start-date 2026-03-01 --end-date 2026-04-16

# 港股 — 腾讯
python src/main.py --tickers 00700.HK --model minimaxai/minimax-m2.7 --analysts-all --start-date 2026-03-01 --end-date 2026-04-16
```

## QVeris Data Source (A-share / HK)

QVeris (`@qverisai/cli`) is installed at `~/.local/bin/qveris`. Credentials are in `ai-hedge-fund/.env` as `QVERIS_API_KEY`.

### Working QVeris Capabilities (global region)

| Capability | QVeris ID | Params | Data |
|---|---|---|---|
| **A股财报三张表** | `ths_ifind.financial_statements.v1` | `codes`, `statement_type`, `year`, `period`, `type` | Income/Balance/CashFlow, 97.5% success |
| A-share daily K (historical) | `mcp_gildata.stockdailyquote.v1` | Natural language `query` | OHLCV, volume, turnover |
| A-share valuation | `mcp_gildata.stockvalueanalysis.v1` | Natural language `query` | PE, PB, PS, EV, PEG, dividend |
| A-share live quote | `hangseng_polysource.a_shares_live_quote.query.v2.10fe0581` | `stockObject: ["公司名"]` | Real-time price |
| HK live quote | `mcp_gildata.hsharelivequote.v1` | Natural language `query` | Real-time price, market cap |
| Research reports | `caidazi.report.query.v1.43d808fc` | `keyword`, `page_size` | Broker research reports |

**Critical: `ths_ifind.financial_statements.v1` is the correct A-share financial data API.** Params example:
```json
{"statement_type":"income","codes":"300750.SZ","year":"2025","period":"1231","type":"1"}
```
- `statement_type`: `income` | `balance` | `cash_flow`
- `period`: `0331`(Q1) | `0630`(H1) | `0930`(Q3) | `1231`(Annual)
- `type`: `1`=Consolidated, `2`=Parent
- Key fields: `ths_revenue_stock`, `ths_np_stock`, `ths_basic_eps_stock`, `ths_total_assets_stock`, `ths_ncf_from_oa_stock`

### Architecture

`src/tools/qveris_api.py` adapter:
1. `is_cn_or_hk_ticker()` detects CN/HK tickers (6-digit for A-share, ends with `.HK`)
2. Calls QVeris CLI: `qveris call <id> --params '...' --json`
3. JSON output wraps data in `result.data` (not top-level)
4. Parses `table_markdown` (Gildata: `|col1|col2|\n|---|\n|val1|val2|`) or nested `data.data.rows` (Polysource)
5. Maps ticker codes to company names for natural-language Gildata queries

`src/tools/api.py` routes: CN/HK → `qveris_api`, US → Financial Datasets API.

### QVeris CLI Quick Reference
```bash
export PATH="$PATH:/opt/data/home/.local/bin"   # qveris is here, NOT ~/.local/bin
export QVERIS_API_KEY=$(grep QVERIS_API_KEY /opt/hermes/ai-hedge-fund/.env | cut -d= -f2)
qveris credits                  # Check balance
qveris discover "A股实时行情"    # Search capabilities
qveris inspect <capability_id>  # View params
qveris call <id> --params '...' --json  # Execute
```

### Hong Kong Stock Workaround (Required!)

**Hedge fund fundamentals/valuation analysts return all N/A for HK stocks** — QVeris financial statement APIs don't support HK tickers. Use direct QVeris CLI queries instead:

```bash
# 1. Real-time quote (works)
qveris call mcp_gildata.hsharelivequote.v1 \
  --params '{"query":"阿里巴巴 09988.HK 实时行情"}' --json

# 2. Research reports (works, richest data source for HK)
qveris call caidazi.report.query.v1.43d808fc \
  --params '{"input":"阿里巴巴 港股 最新研报","page_size":5}' --json

# 3. HK financial statements (BROKEN — returns 404)
# hangseng_polysource.financialstatement.query.v2.* — NOT working on global region

# 4. HK valuation analysis (BROKEN — returns empty)
# mcp_gildata.stockvalueanalysis.v1 — empty for HK stocks
```

**Recommended HK analysis approach**: Combine real-time price quote + research reports for manual synthesis. Research reports contain: revenue/profit forecasts, segment breakdowns, target prices, analyst ratings, and strategic commentary.

### Known Limitations
- **HK stock financial statements**: `hangseng_polysource.financialstatement.query.v2.*` returns 404 on global region — NO workaround via QVeris
- **HK stock valuation**: `mcp_gildata.stockvalueanalysis.v1` returns empty table for HK tickers
- **Hedge fund on HK**: Fundamentals/valuation/technical analysts all return N/A; only news_sentiment gives a result (neutral with 0 articles). Portfolio manager outputs "hold" with no valid trade
- `hangseng_polysource.stock.*` (financial_data_comparison, basicCorpInfo) returns 404 on global region
- `mcp_gildata.financialstatement.v1` returns empty — use `ths_ifind.financial_statements.v1` instead (A-share only)
- No A-share insider trades or news via QVeris yet
- Each QVeris call costs 1-3 credits

## Non-US Markets: Additional Alternatives

The `Financial Datasets API` **only covers US publicly-traded companies** (confirmed via [official docs](https://docs.financialdatasets.ai/market-coverage): "Non-US markets are not yet available."). Korean (`.KS`), Hong Kong (`.HK`), and A-shares (`.SS`/`.SZ`) all return empty data.

For A股/港股/美股 analysis with Chinese-language queries, consider **同花顺问财** via `mcp_query_table`:

```bash
pip install mcp_query_table
# Requires Chrome with --remote-debugging-port=9222
python -m mcp_query_table --format markdown --endpoint http://127.0.0.1:9222
```

This is an MCP server that uses Playwright to query iwencai.com with natural language (e.g., "年初至今涨幅最大的50只港股"). Can be added to Hermes via native MCP client.

## Known Bugs & Fixes

### `.SS` → `.SH` ticker normalization (CRITICAL)
`_ticker_with_exchange()` in `qveris_api.py` was returning `.SS` (Yahoo Finance format) but iFinD API requires `.SH`. This caused ALL `ths_ifind.financial_statements.v1` calls to fail with "API error: error happen with input parameters". **Fix**: normalize `.SS` → `.SH` before returning:
```python
if t.endswith(".SS"):
    return t[:-3] + ".SH"
```

### `currency` field missing from FinancialMetrics (CRITICAL)
`FinancialMetrics` is a Pydantic model that requires `currency`. The QVeris adapter only set it inside conditional blocks (when Gildata/iFinD calls succeeded), so any API failure left it unset, crashing with `ValidationError: currency Field required`. **Fix**: set `currency="CNY"` (or `"HKD"`) as default in the initial `metrics_kwargs` dict, not inside conditional blocks.

### `gross_margin` calculation used wrong field
Was using `ths_operating_total_cost_stock` (total operating cost including R&D, SGA) instead of `ths_operating_cost_stock` (COGS) for gross margin. Gross margin = `(revenue - COGS) / revenue` where COGS = `ths_operating_cost_stock`. Also `operating_margin` and `net_margin` must handle negative values correctly (they can be < 0 for unprofitable companies).

### `search_line_items_qveris` field mapping bugs
`gross_profit` was mapped to `ths_op_stock` (operating profit) instead of computing `revenue - COGS`. Fix: compute after field mapping:
```python
rev = _safe_float(row.get("ths_revenue_stock"))
cogs = _safe_float(row.get("ths_operating_cost_stock"))
if rev is not None and cogs is not None:
    mapped["gross_profit"] = rev - cogs
```
Also added `total_equity` = `total_assets - total_liabilities` computation.

### Missing derived metrics in `get_financial_metrics_qveris`
The adapter was not computing: ROE, ROA, debt_to_equity, quick_ratio (was using current_ratio as approximation), asset_turnover, inventory_turnover, receivables_turnover, interest_coverage, book_value_per_share, revenue_growth (YoY), earnings_growth (YoY), book_value_growth (YoY). These require fetching both current and prior year statements. The rewritten function fetches 2 years of data and computes all derived metrics.

### `caidazi.report.query.v1` needs `input` parameter
The research reports API returns "缺少必需的查询参数: 'input'" — it requires an `input` field, not `keyword`. Fix the call:
```python
data = _qveris_call("caidazi.report.query.v1.43d808fc",
                    {"input": f"{name} {code} 最新研报", "page_size": limit})
```

### Running non-interactively: bypass CLI entirely
The CLI's `questionary` prompts hang in non-TTY contexts even with `--analysts-all`. Alternative: call `run_hedge_fund()` directly via Python subprocess:
```python
from src.main import run_hedge_fund
result = run_hedge_fund(
    tickers=[ticker], start_date=..., end_date=...,
    portfolio=..., show_reasoning=False,
    selected_analysts=['cathie_wood', 'fundamentals_analyst', ...],
    model_name='minimaxai/minimax-m2.7', model_provider='OpenAI',
)
```
Run inside venv: `cd /opt/hermes/ai-hedge-fund && source .venv/bin/activate && python3 -c "..."`

### QVeris credits burn fast
Each analysis run with 7 analysts makes ~30-50 API calls (3-4 per analyst for financials + valuation + price + news). At 1-3 credits each, a single run costs 50-150 credits. Check balance with `qveris credits` before running. The free tier (~2000 credits) supports ~15-20 full analyses.

## Pitfalls

- **Interactive prompt hangs**: The `--analysts-all` flag does NOT exist in current code. Use direct Python `run_hedge_fund()` call instead
- **Import error**: Set `PYTHONPATH=/opt/hermes/ai-hedge-fund` before running
- **Non-US tickers**: Financial Datasets API doesn't support them — all data returns null (confirmed 07709.HK, 000660.KS)
- **Rate limits**: NVIDIA NIM free tier may return 429 on rapid sequential calls; wait 60s+
- **Cold start**: Large models (70B+) take 15-30s on first request
- **FINANCIAL_DATASETS_API_KEY**: Check `.env` — must be set or fundamentals data is empty
- **QVeris ticker detection**: `is_cn_or_hk_ticker()` in `src/tools/qveris_api.py` — 6-digit codes or `.HK` suffix route to QVeris
- **QVeris JSON parsing**: Output is `{ "result": { "data": { ... } } }` — extract `result.data`, not top level
- **Gildata table parsing**: Responses use markdown table format — `_parse_gildata_table()` splits `|col|` rows
- **Credits exhaustion produces MISLEADING neutral signals** — When QVeris credits run out mid-run, analysts don't crash visibly. They return "neutral" signals with reasoning like "insufficient data", "missing financials", or "FCF unavailable". This looks like a data quality issue but is actually `CREDITS_INSUFFICIENT` API errors. **Always check `qveris credits` before and after runs.** If you see multiple analysts returning neutral/empty for the same stock, suspect credits exhaustion first.

- **Hermes config for custom models**: Add to `custom_providers` in `/opt/data/config.yaml`:
  ```yaml
  custom_providers:
  - name: NVIDIA NIM
    base_url: https://integrate.api.nvidia.com/v1
    api_key: nvapi-YOUR_KEY
    model: minimaxai/minimax-m2.7
  ```
  This adds the model as a selectable provider in Hermes without changing the default model.
