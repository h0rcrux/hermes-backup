---
name: siyuan-shared-memory
description: 思源笔记共享记忆 — Hermes/OpenClaw 双Agent共享记忆读写
tags: [memory, siyuan, shared, collaboration]
---

# 思源笔记共享记忆

## 用途
Hermes 和 OpenClaw 双 Agent 共享记忆系统。通过思源 REST API 读写同一个笔记本。

## 配置
- 地址: http://192.168.31.123:6806
- Token: f4ru7b89hxq9krj1
- 笔记本: Agent-Shared-Memory (id=20260421142356-5vz0067)
- Adapter 本地: /opt/hermes/siyuan-adapter/siyuan.py
- Adapter 在思源中: `🔧 工具与环境/Siyuan Adapter`（OpenClaw 从这里取）
- 每日同步 Cron: `0 1 * * *` (job_id=ebd4fc29deb8)

## 知识库结构
```
README                          → 开机指南 + 每日同步规则
🧑‍💼 用户画像/基本信息           → Jeff 的信息（agent 只读）
🧠 跨会话记忆/YYYY-MM-DD         → 每日记忆（append）
🧠 跨会话记忆/决策记录           → 重要决策 + 理由
🧠 跨会话记忆/经验教训           → 犯过的错
📚 知识库/投资笔记, 项目笔记, 读书笔记
🔧 工具与环境/环境配置, API密钥索引, 常用命令, Siyuan Adapter
📋 待办与计划/本周计划, 长期目标
```

## 用法

### Python
```python
import sys
sys.path.insert(0, "/opt/hermes/siyuan-adapter")
from siyuan import SharedMemory

mem = SharedMemory()

# 读取（Markdown 格式，已清理 kramdown 标记）
content = mem.read_md("🧑‍💼 用户画像/基本信息")

# 写入（v2: 文档已存在则自动追加，不会覆盖）
mem.write("📚 知识库/投资笔记/茅台", "分析内容...", agent="Hermes")

# 追加到指定文档
mem.append("🧠 跨会话记忆/决策记录", "## 新决策...")

# 每日记录（自动创建或追加到当日文档）
mem.log("完成了xxx", agent="Hermes")

# 搜索
results = mem.search("投资")

# 列出
docs = mem.list_docs("📚 知识库")
```

### CLI
```bash
python3 /opt/hermes/siyuan-adapter/siyuan.py md <路径>      # 读取Markdown
python3 /opt/hermes/siyuan-adapter/siyuan.py log <内容>     # 记录到今日
python3 /opt/hermes/siyuan-adapter/siyuan.py search <关键词>
python3 /opt/hermes/siyuan-adapter/siyuan.py list [前缀]
```

## 安全策略 (v2)
| 方法 | 行为 | 安全性 |
|------|------|--------|
| read / read_md / search | 只读 | ✅ 安全 |
| log | appendBlock 追加 | ✅ 安全 |
| append | appendBlock 追加 | ✅ 安全 |
| write(path, content) | 已存在则追加 | ✅ 安全 |
| write(path, content, append_if_exists=False) | 删除重建 | ⚠️ 仅限独占时 |

**核心规则：两个 agent 写同一篇文档，内容追加，不互相覆盖。**

## 读写权限
- 🧑‍💼 用户画像 → 用户手动改，agent 只读
- 🧠 跨会话记忆 → 两个 agent 都能追加，不改历史
- 📚 知识库 → 谁产出谁写，标记来源 agent
- 🔧 工具环境 → 谁发现谁写，更新时标记时间
- 📋 待办 → 两个 agent 都能读写

## 每日同步规则
每天 1:00 AM 自动同步（cron job 已建好）：
1. 读取 `🧠 跨会话记忆` 最近 7 天的记录
2. 读取 `📋 待办与计划/本周计划`，检查进度
3. 写入当天记录到 `🧠 跨会话记忆/YYYY-MM-DD`
4. 如有需要更新待办

## ⚠️ 思源 API 踩坑记录

### getDoc 的 notebook+path 模式不可靠
```python
# ❌ 这个会返回 data: null
api("/api/filetree/getDoc", {"notebook": nb_id, "path": "/🧑‍💼 用户画像/基本信息", "mode": 0})

# ✅ 用 ID 才行
api("/api/filetree/getDoc", {"id": doc_id})
```

### 用 SQL 查文档 ID
```python
r = api("/api/query/sql", {
    "stmt": f"SELECT id FROM blocks WHERE box='{nb_id}' AND type='d' AND hpath='/路径' LIMIT 1"
})
doc_id = (r.get("data") or [{}])[0].get("id")
```

### createDocWithMd 返回格式
```python
# 返回 data 是字符串（ID），不是对象
{"code": 0, "msg": "", "data": "20260421142409-zk8zfl5"}
# 不是 {"data": {"id": "..."}}
```

### getBlockKramdown 需要清理
返回的 kramdown 包含思源标记，需要正则清理：
```python
# 清理 {: id="xxx" updated="xxx" } 标记行
line = re.sub(r'\s*\{:\s*id="[^"]*"[^}]*\}', '', line)
# 清理末尾的 doc 元数据块
result = re.sub(r'\n*\{:\s*title="[^"]*"\s*type="doc"[^}]*\}\s*$', '', result)
```

### search API 返回重复结果
```python
# 需要按 hpath 去重
seen = set()
for item in blocks:
    hpath = item.get("hPath", "")
    if hpath not in seen:
        seen.add(hpath)
        results.append(item)
```

### listDocPaths 端点不存在
用 SQL 替代：
```python
api("/api/query/sql", {"stmt": f"SELECT hpath, content FROM blocks WHERE box='{nb_id}' AND type='d' ORDER BY hpath"})
```

### 不要用 curl，用 Python urllib
本机没有安装 curl。所有 API 调用用 Python 内置 urllib：
```python
import urllib.request, json

def api(url, body=None):
    data = json.dumps(body or {}).encode()
    req = urllib.request.Request(url, data=data, headers={
        "Authorization": "Token <token>",
        "Content-Type": "application/json"
    }, method="POST")
    return json.loads(urllib.request.urlopen(req, timeout=10).read())
```

### API 返回 data 可能是 None
所有 API 返回值的 `data` 字段可能为 `None`，必须防御性取值：
```python
# ❌ 会报 AttributeError: 'NoneType' object has no attribute 'get'
value = r.get("data", {}).get("id")

# ✅ 安全写法
value = (r.get("data") or {}).get("id")
```

### 读取其他笔记本（非共享记忆）
SharedMemory 类默认只操作 Agent-Shared-Memory 笔记本。要读取其他笔记本，直接用 API：

```python
import urllib.request, json

TOKEN = "f4ru7b89hxq9krj1"
BASE = "http://192.168.31.123:6806"

def api(url, body=None):
    data = json.dumps(body or {}).encode()
    req = urllib.request.Request(f"{BASE}{url}", data=data, headers={
        "Authorization": f"Token {TOKEN}",
        "Content-Type": "application/json"
    }, method="POST")
    return json.loads(urllib.request.urlopen(req, timeout=10).read())

# 1. 列出所有笔记本，找到目标
r = api("/api/notebook/lsNotebooks")
notebooks = r.get("data", {}).get("notebooks", [])
target_nb = next((nb for nb in notebooks if nb["name"] == "量化交易平台"), None)
nb_id = target_nb["id"]

# 2. 查询该笔记本下的所有文档
r = api("/api/query/sql", {
    "stmt": f"SELECT hpath FROM blocks WHERE box='{nb_id}' AND type='d' ORDER BY hpath"
})
docs = r.get("data", [])

# 3. 读取单篇文档内容（HTML格式）
r = api("/api/query/sql", {
    "stmt": f"SELECT id FROM blocks WHERE box='{nb_id}' AND type='d' AND hpath='/提示词1-搭建项目骨架' LIMIT 1"
})
doc_id = (r.get("data") or [{}])[0].get("id")
r = api("/api/filetree/getDoc", {"id": doc_id})
html_content = r.get("data", {}).get("content", "")

# 4. HTML → 纯文本
import re
text = re.sub(r'<[^>]+>', '', html_content)
text = text.replace('&amp;', '&').replace('&lt;', '<').replace('&gt;', '>').replace('&#39;', "'")
```

**关键点：**
- `read_md()` / `list_docs()` 等方法只操作默认笔记本
- 跨笔记本读取必须用原始 API + SQL 查询
- getDoc 返回 HTML，需要自行清理标签
- 批量读取时先查 hpath 列表，再逐个读取，避免遗漏

### OpenClaw 接入方式
不需要复制文件。把 adapter 源码写入思源的 `🔧 工具与环境/Siyuan Adapter` 文档，OpenClaw 从思源读取源码保存到本地即可使用。README 文档有 3 步入门指南。
