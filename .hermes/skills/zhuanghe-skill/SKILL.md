---
name: zhuanghe-skill
version: 3.2
description: "A股股票分析技能 — 基于 AkShare 的五维分析（技术面/基本面/资金面/消息面/形态面），支持K线形态识别(60+)、缠论买卖点、信号共振评分、多视角投票。"
author: "zhuang-HE"
homepage: https://github.com/zhuang-HE/stock-analyst-skill
tags:
  - 股票
  - A股
  - 技术分析
  - 基本面
  - 形态识别
  - 缠论
  - 资金流向
created: "2026-04-17"
---

# zhuanghe-skill — A股股票分析

基于 AkShare 开源金融数据库的五维股票分析工具。

## 触发条件

当用户要求分析 A股股票、查看行情、做技术分析、看资金流向时使用此技能。

## 环境

- 项目路径: `/opt/hermes/stock-analyst-skill/`
- 虚拟环境: `/opt/hermes/.venv/` (**不是**项目目录下的 .venv)
- CLI 入口: `~/.local/bin/zhuanghe`
- 上游仓库: https://github.com/zhuang-HE/stock-analyst-skill

## 快速使用

```bash
source /opt/hermes/.venv/bin/activate
python3 /opt/hermes/stock-analyst-skill/scripts/full_analysis.py <股票代码>
```

CLI 快捷方式:
- `zhuanghe 603986` — JSON 输出
- `zhuanghe 603986 --text` — Markdown 报告

## API 命名陷阱（已踩坑）

上游代码的 README 和实际实现有差异：

| README 写的 | 实际方法名 | 位置 |
|:---|:---|:---|
| `recognizer.recognize(df)` | `recognizer.recognize_all(df)` | patterns/candlestick.py |
| `ChanLunAnalyzer()` | `ChanlunAnalyzer()` | patterns/chanlun.py |
| `StockFullAnalyzer.analyze(code)` | `StockFullAnalyzer.get_full_analysis(code)` | scripts/full_analysis.py |
| `report['text']` | 只有 `report['markdown']` | templates/unified_report_template.py |

## 核心 API

```python
# 完整分析（返回 dict）
from scripts.full_analysis import StockFullAnalyzer
result = StockFullAnalyzer().get_full_analysis('603986')

# K线形态识别（60+种）
from patterns.candlestick import CandlestickPatternRecognizer
recognizer = CandlestickPatternRecognizer()
patterns = recognizer.recognize_all(df)

# 缠论分析（笔/中枢/买卖点）
from patterns.chanlun import ChanlunAnalyzer
result = ChanlunAnalyzer().analyze(df)

# 信号共振评分
from signals.scoring import SignalResonanceScorer, Signal, SignalType, SignalDirection
resonance = SignalResonanceScorer().calculate_resonance(signals)

# 生成报告
from templates.unified_report_template import UnifiedReportGenerator
report = UnifiedReportGenerator(data).generate_unified_report()

# 多视角投票
from multi_perspective import MultiPerspectiveAnalyzer
```

## 数据源限制

- **A股**: 完整支持（行情/财务/资金/新闻）
- **港股**: 部分支持（新闻可用，行情/财务报错）
- **AkShare 网络**: 偶有不稳定，建议重试 2-3 次

## 依赖

已安装在 `/opt/hermes/.venv/`: akshare, pandas, numpy, cairosvg
