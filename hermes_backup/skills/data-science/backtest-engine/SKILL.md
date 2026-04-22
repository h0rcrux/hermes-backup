---
name: backtest-engine
description: Python 回测引擎 - 放入 QMT 导出的 CSV 数据即可自动跑策略回测
category: data-science
---

# 回测引擎技能

## 目录结构

```
/opt/hermes/backtest/
├── engine.py          # 回测引擎（主程序）
├── qmt_export.py      # QMT 数据导出脚本（Windows 运行）
├── data/              # 📥 用户放入 CSV 的目录
├── output/            # 📤 回测结果输出目录
└── strategies/        # 自定义策略目录
    └── _example.py    # 策略模板
```

## 触发条件

当用户提到：
- "跑回测"、"回测一下"、"backtest"
- 提供了 CSV 文件或提到 data/ 目录
- 想测试某个策略

## 操作流程

1. 检查 `/opt/hermes/backtest/data/` 是否有新数据
2. 运行引擎：`source /opt/hermes/ai-hedge-fund/.venv/bin/activate && cd /opt/hermes/backtest && python engine.py --all`
3. 读取 `output/` 中的结果报告
4. 给用户汇总分析

## 数据文件要求

### 文件名格式（自动检测股票代码）
- `688110.csv` → 688110.SH
- `300750.SZ.csv` → 300750.SZ
- `688110_daily.csv` → 688110.SH

### 支持的列名（自动映射）
- 中文：开盘/开盘价、最高/最高价、最低/最低价、收盘/收盘价、成交量、成交额
- 英文：open、high、low、close、volume、amount
- 日期列：时间/日期/date/trade_date

### 支持格式
- CSV（自动检测编码 utf-8/gbk/gb2312）
- Excel（.xlsx/.xls）
- Parquet

## 运行命令

```bash
# 激活环境
source /opt/hermes/ai-hedge-fund/.venv/bin/activate
cd /opt/hermes/backtest

# 列出所有可用数据和策略
python engine.py --list

# 对所有数据跑所有策略
python engine.py --all

# 指定策略
python engine.py -s ma_cross -t 688110.SH

# 多策略对比
python engine.py -s ma_cross,rsi,macd,boll,kdj,turtle --all

# 指定时间段和资金
python engine.py -s ma_cross --start 2024-01-01 --end 2025-12-31 --cash 200000
```

## 内置策略

| 策略名 | 说明 | 核心参数 |
|---|---|---|
| ma_cross | 双均线交叉 | fast=10, slow=30 |
| rsi | RSI 超买超卖 | period=14, oversold=30, overbought=70 |
| macd | MACD 金叉死叉 | 12/26/9 |
| boll | 布林带突破 | period=20, std_mult=2.0 |
| kdj | KDJ 金叉 | n=9 |
| turtle | 海龟交易法则 | entry=20, exit=10 |

## 添加自定义策略

在 `/opt/hermes/backtest/strategies/` 下创建 `.py` 文件：

```python
DESC = "我的策略描述"

def strategy(df, i, positions, date, **kwargs):
    """
    df: K线 DataFrame (index=date, columns=open/high/low/close/volume)
    i: 当前 K线序号
    positions: 持仓 {'688110.SH': {'shares': 100, 'cost': 50.0}}
    date: 当前日期
    返回: 'BUY' / 'SELL' / None
    """
    if i < 20:
        return None
    close = df['close'].values
    ma5 = close[i-5:i].mean()
    ma20 = close[i-20:i].mean()
    ticker = kwargs.get('ticker', 'stock')
    has = ticker in positions and positions[ticker]['shares'] > 0
    if ma5 > ma20 and not has:
        return 'BUY'
    if ma5 < ma20 and has:
        return 'SELL'
```

## 回测参数

- 初始资金：默认 100,000
- 佣金：万3（0.03%）
- 印花税：千1（0.1%，仅卖出）
- 最小交易：100 股
- 买入仓位：默认 25% 可用资金

## 输出文件

每次回测生成 3 个文件在 `output/`：
- `{ticker}_{strategy}_{timestamp}_report.json` — 回测报告
- `{ticker}_{strategy}_{timestamp}_daily.csv` — 每日净值曲线
- `{ticker}_{strategy}_{timestamp}_trades.csv` — 交易记录

## 输出指标

- 总收益率 / 年化收益
- 基准收益（买入持有对比）
- 超额收益
- 最大回撤 / 回撤日期
- Sharpe Ratio / Sortino Ratio
- 胜率 / 盈亏比 / 平均盈亏
- 交易明细

## QMT 数据导出

用户在 Windows miniQMT 中运行 `qmt_export.py`：

```python
from xtquant import xtdata
xtdata.download_history_data('688110.SH', period='1d', start_time='2023-01-01')
df = xtdata.get_market_data_ex(
    field=['open','high','low','close','volume','amount'],
    stock_list=['688110.SH'], period='1d', start_time='2023-01-01')
df['688110.SH'].to_csv('D:\\688110_daily.csv')
```

然后上传到服务器：`scp D:\688110_daily.csv root@server:/opt/hermes/backtest/data/`
