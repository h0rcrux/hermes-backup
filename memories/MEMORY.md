OpenClaw×Hermes互保+共享记忆：部署脚本/opt/hermes/repair/，需SSH进NAS重建容器。思源共享记忆v2：NAS 192.168.31.123:6806，Token=f4ru7b89hxq9krj1，笔记本Agent-Shared-Memory，adapter /opt/hermes/siyuan-adapter/siyuan.py，skill: siyuan-shared-memory。每日1:00AM同步(cron ebd4fc29deb8)。飞书：ZQBmdvtStorDftxXCzFc8LFZnMd
§
User has lark-cli configured with app ID cli_a9574160a1385bc6. Auth tokens expire ~2h (user) / 7d (refresh). User Jeff (ou_61b0f96337af7b4e3569203724e8b5b6).
§
AI Hedge Fund at /opt/hermes/ai-hedge-fund. NVIDIA NIM (minimaxai/minimax-m2.7). venv at .venv/. QVeris adapter fully fixed: .SS→.SH, currency default, gross_margin=rev-COGS, ROE/ROA/growth, caidazi `input` param. Backtest engine built at /opt/hermes/backtest/ (skill: backtest-engine).
§
Hermes config at /opt/data/config.yaml. Add custom LLM providers under `custom_providers:` list (name, base_url, api_key, model). Current default: xiaomi/mimo-v2-pro via nous.
§
QVeris CLI installed at ~/.local/bin/qveris (added to PATH). API key: sk-Zp2hmLebDtg7WTy76vWgJhgvsdCZFP5Nw3MTaGzZCJw. Region: global. Credits EXHAUSTED (0 remaining) — full hedge fund run with 6-7 analysts costs ~80-150 credits/stock. Need recharge at qveris.ai/pricing. Key APIs: ths_ifind.financial_statements.v1 for A-share financials, mcp_gildata.stockvalueanalysis.v1 for valuation. caidazi.report.query.v1 param is `input` not `keyword`. SS→SH conversion required for iFinD.
§
Hermes config (/opt/data/config.yaml) custom_providers: NVIDIA NIM (minimaxai/minimax-m2.7, working), NVIDIA NIM (Kimi k2.5, 404 broken), Api.date-now.uk (gpt-5.4, timeout). Default: xiaomi/mimo-v2-pro via nous (401 key issue).
§
zhuanghe-skill: A股五维分析. /opt/hermes/stock-analyst-skill/, venv /opt/hermes/.venv/, upstream github.com/zhuang-HE/stock-analyst-skill. CLI: `zhuanghe <code>`. API gotchas: .recognize_all() not .recognize(), ChanlunAnalyzer not ChanLun, get_full_analysis(code), report only 'markdown' key. A-shares only, HK partial. AkShare unstable.
§
SOUL.md人格：段永平（大道无形我有型）。投资+书单已融入。文档：investment-philosophy.md, reading-list.md（均在/opt/data/）