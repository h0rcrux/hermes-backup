---
name: qveris
description: QVeris capability routing network — discover, inspect, and call 10,000+ real-world capabilities through one CLI/API. Covers A股/港股/美股 data, research reports, valuation, and more.
tags: [finance, api, data, a-share, hk-stock, qveris, ifind]
---

# QVeris — Capability Routing Network

QVeris is a capability routing network for AI agents. Instead of hard-coding individual API integrations, agents use QVeris to discover, inspect, and call real-world capabilities through one unified protocol.

## Core Protocol: Discover → Inspect → Call

| Step | Command | Cost | Purpose |
|------|---------|------|---------|
| **Discover** | `qveris discover "..."` | Free | Find capabilities by natural language |
| **Inspect** | `qveris inspect <id>` | Free | View params, schema, success rate, latency |
| **Call** | `qveris call <id> --params '{...}'` | 1-100 credits | Execute and get structured JSON |

## Setup

```bash
npm install -g @qverisai/cli
qveris config set api_key YOUR_KEY
qveris doctor   # verify
```

**Region:** `sk-xxx` → Global (qveris.ai), `sk-cn-xxx` → China (qveris.cn)

## Critical: How to Write Discover Queries

### Rule 1: Describe the TOOL TYPE, not the data

| ❌ Bad | ✅ Good |
|--------|---------|
| `"宁德时代 latest earnings"` | `"financial statement API"` |
| `"腾讯 stock price today"` | `"stock quote real-time API"` |
| `"A股涨幅最大的股票"` | `"stock market gainers ranking API"` |

**Discover answers "which API tool can do X?" — not "what is the value of Y?"**

### Rule 2: Write queries in ENGLISH

Even when querying Chinese stock data:
- `"港股实时行情"` → `"Hong Kong stock real-time quote API"`
- `"A股财务报表"` → `"A-share financial statement API"`

### Rule 3: Retry with rephrased queries if results are poor

## CLI Flags

```bash
qveris --json              # Structured JSON output
qveris --dry-run           # Validate without credits
qveris --codegen python    # Generate code snippet
qveris inspect 1           # Session index shortcut
qveris call 2              # Session index shortcut
```

## Verified A-Share / HK Stock APIs

### Financial Statements (Three Statements) ⭐ PRIMARY
```
ID: ths_ifind.financial_statements.v1
Provider: 同花顺 iFinD  |  Success: 97.5%  |  ~267ms
Params:
  codes: "300750.SZ"          (comma-separated, max 50)
  statement_type: "income" | "balance" | "cash_flow"
  year: "2025"
  period: "1231" (Annual) | "0630" (H1) | "0331" (Q1) | "0930" (Q3)
  type: "1" (Consolidated)
```

### Also available individually (iFinD)
- `ths_ifind.income_statement.v1` — Income statement only (92.3%)
- `ths_ifind.balance_sheet.v1` — Balance sheet only (88.9%)
- `ths_ifind.cash_flow_statement.v1` — Cash flow only (97.2%)

### Valuation Indicators
```
ID: mcp_gildata.stockvalueanalysis.v1
Params: {"query": "宁德时代 300750 最新估值指标"}
Returns: PE(TTM), PB, PS(TTM), EV/EBITDA, PEG, dividend yield, market cap (万元)
```

### A-Share Daily OHLCV (Historical)
```
ID: mcp_gildata.stockdailyquote.v1
Params: {"query": "宁德时代 300750 2026年3月到4月日K数据"}
Returns: open, high, low, close, volume, turnover rate
```

### A-Share Real-Time Quote
```
ID: hangseng_polysource.a_shares_live_quote.query.v2.10fe0581
Params: {"stockObject": ["宁德时代"]}  (Chinese name)
⚠️ May return 404 on Global region
```

### HK Stock Real-Time Quote
```
ID: mcp_gildata.hsharelivequote.v1
Params: {"query": "腾讯控股 00700.HK 实时行情"}
Returns: price, volume, market cap, PE, PB
```

### Research Reports
```
ID: caidazi.report.query.v1.43d808fc
Params: {"keyword": "宁德时代 最新研报", "page_size": 10}
```

### Other iFinD APIs
- `ths_ifind.money_flow.v1` — Money flow (77.5%)
- `ths_ifind.share_pledge_stock.v1` — Share pledge
- `ths_ifind.market_breadth.v1` — Market breadth
- `ths_ifind.top_movers.v1` — Gainers/losers ranking
- `ths_ifind.hk_connect_stats.v1` — 沪深港通 stats

## iFinD Financial Statement Fields

### Income Statement (补充完整字段)
| Field | Meaning |
|-------|---------|
| `ths_revenue_stock` | 营业收入 |
| `ths_operating_cost_stock` | 营业成本 (COGS) — 用于计算毛利 |
| `ths_operating_total_cost_stock` | 营业总成本 (含费用，非毛利用) |
| `ths_op_stock` | 营业利润 |
| `ths_np_stock` | 净利润 |
| `ths_basic_eps_stock` | 基本每股收益 |
| `ths_rad_cost_sum_stock` | 研发费用 |
| `ths_total_profit_stock` | 利润总额 |
| `ths_financing_expenses_stock` | 财务费用 |
| `ths_sales_fee_stock` | 销售费用 |
| `ths_manage_fee_stock` | 管理费用 |

### Balance Sheet (补充完整字段)
| Field | Meaning |
|-------|---------|
| `ths_total_assets_stock` | 总资产 |
| `ths_total_liab_stock` | 总负债 |
| `ths_total_current_assets_stock` | 流动资产 |
| `ths_total_current_liab_stock` | 流动负债 |
| `ths_currency_fund_stock` | 货币资金 |
| `ths_inventory_stock` | 存货 |
| `ths_account_receivable_stock` | 应收账款 |

### Cash Flow (补充完整字段)
| Field | Meaning |
|-------|---------|
| `ths_ncf_from_oa_stock` | 经营活动现金流净额 |
| `ths_ncf_from_ia_stock` | 投资活动现金流净额 |
| `ths_ncf_from_fa_stock` | 筹资活动现金流净额 |
| `ths_final_balance_of_cce_stock` | 期末现金余额 |

### Ticker Format Rules
| 格式 | 示例 | 说明 |
|------|------|------|
| Yahoo `.SS` | `603986.SS` | 必须转换为 `.SH` 再调 iFinD |
| iFinD `.SH` | `603986.SH` | 上交所正确格式 |
| `.SZ` | `300750.SZ` | 深交所，两种格式一致 |
| `.HK` | `00700.HK` | 港股，两种格式一致 |

## Pricing

- Free: 1,000 signup + 100 daily credits
- Pro: $19/mo = 10,000 credits (never expire)
- Typical call: 1-3 credits

## Pitfalls

1. **Discover query must be English + tool type description** — NOT specific data entities
2. **hangseng_polysource APIs** return 404 on Global region — may need China key
3. **`mcp_gildata.financialstatement.v1` is WRONG** — use `ths_ifind.financial_statements.v1`
4. **Natural language query params** (mcp_gildata.*) use Chinese; discover queries use English
5. **iFinD params are strict** — all 5 required fields must be present
6. **Credits drain fast** — 20 agents × multiple calls = ~30 credits per stock analysis; ai-hedge-fund full run with 6-7 analysts can burn 80-150 credits
7. **iFinD year fallback** — Annual reports may not be filed yet for current year; always try `year=N` then `year=N-1`
8. **iFinD null data check** — When year data isn't available yet, all fields return null; check with `any(v is not None for k, v in row.items() if k not in ("statement_type","thscode","time"))` before processing
9. **CLI output nesting** — `qveris --json` returns `{result: {data: ...}}`; unwrap with `full.get("result", {}).get("data", full)` before processing
10. **`.SS` → `.SH` ticker conversion** — Yahoo Finance format uses `.SS` for Shanghai, but iFinD requires `.SH`. Must normalize before calling `ths_ifind.*` APIs. `.SZ` for Shenzhen is the same in both.
11. **`caidazi.report.query.v1` param is `input` not `keyword`** — The research report API requires `{"input": "..."}`, not `{"keyword": "..."}`. Using `keyword` returns "缺少必需的查询参数: 'input'".
12. **Credits exhausted = exit code 77** — When credits are depleted, CLI returns exit code 77 with `"code": "CREDITS_INSUFFICIENT"`. The Python adapter's `_qveris_call` function checks `returncode != 0` and returns None — downstream code must handle this gracefully (don't crash on None data).
13. **Currency field is required in Pydantic models** — When building adapters for frameworks like ai-hedge-fund, always set `currency` in the initial dict defaults (e.g., `currency="CNY"`), not conditionally inside API response blocks. If the API call fails, the model validation will crash without it.
14. **Gross margin = revenue - COGS, not revenue - total_cost** — `ths_operating_cost_stock` is COGS; `ths_operating_total_cost_stock` includes SG&A, R&D, etc. Use `ths_operating_cost_stock` for gross margin calculation.

## Integrating QVeris with Python Projects

### CLI output parsing
```python
import json, subprocess

def qveris_call(capability_id, params, timeout=60):
    cmd = ["qveris", "call", capability_id, "--params", json.dumps(params), "--json"]
    r = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
    full = json.loads(r.stdout)
    if not full.get("success", True):
        return None
    return full.get("result", {}).get("data", full)
```

### Three response formats to handle
1. **Gildata table_markdown** (`mcp_gildata.*`) — `data.results[0].table_markdown` is a markdown table; parse header row + data rows
2. **iFinD structured** (`ths_ifind.*`) — `data` is a list of lists; `data[0][0]` is the record dict with `ths_*` prefixed fields
3. **Polysource nested** (`hangseng_polysource.*`) — `data.data.rows` or `data.rows`; ⚠️ may return 404 on Global region

### Year fallback pattern for iFinD
```python
for yr in [current_year, str(int(current_year) - 1)]:
    data = qveris_call("ths_ifind.financial_statements.v1", {
        "codes": "300750.SZ", "statement_type": "income",
        "year": yr, "period": "1231", "type": "1",
    })
    if data and data[0] and any(v is not None for k, v in data[0][0].items()
                                  if k not in ("statement_type","thscode","time")):
        # Got valid data, use it
        break
```

### ai-hedge-fund integration
When integrating with projects using Pydantic `LineItem(extra="allow")`, add `__getattr__` to allow dot-access to extra fields:
```python
def __getattr__(self, name):
    try:
        extras = object.__getattribute__(self, "model_extra")
        if extras and name in extras:
            return extras[name]
    except AttributeError:
        pass
    return None  # Return None for missing fields instead of raising
```
