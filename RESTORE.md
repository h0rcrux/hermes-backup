# Hermes 备份恢复指南

## 备份内容（2026-04-22）

- `config.yaml` — API Key + 模型配置
- `.env` — 各平台 Token
- `SOUL.md` — Agent 人格（段永平哲学）
- `memories/` — MEMORY.md + USER.md（跨会话记忆）
- `skills/` — 27 个分类，89 个 Skill
- `home/.hermes/` — 思源共享记忆适配器
- `run_agent.py` — 免疫系统代码修改

## 恢复步骤

### 前提：先在新 Docker 容器里把 Hermes 跑起来

```bash
# 1. 克隆 Hermes（如果你还没有）
git clone https://github.com/aor夫人的/hermes-agent.git
cd hermes-agent
docker compose up -d

# 2. 进入容器
docker compose exec hermes bash

# 3. 把备份解压到 /opt/data/
# 方法A：如果备份文件在宿主机（挂载的 /opt/data 共享）
cp /path/to/hermes-backup-complete.tar.gz /opt/data/
cd /opt/data
tar -xzf hermes-backup-complete.tar.gz
rm hermes-backup-complete.tar.gz

# 方法B：如果备份在容器内
# 备份文件已经在 /opt/data/ 下了，直接解压
cd /opt/data
tar -xzf hermes-backup-complete.tar.gz
rm hermes-backup-complete.tar.gz

# 4. 重启 Hermes
hermes restart
```

## 数据覆盖说明

- `config.yaml` → `/opt/data/config.yaml`（覆盖，含新 API Key）
- `.env` → `/opt/data/.env`（覆盖）
- `SOUL.md` → `/opt/data/SOUL.md`（覆盖）
- `memories/` → `/opt/data/memories/`（覆盖）
- `skills/` → `/opt/data/skills/`（合并，不冲突的保留）
- `run_agent.py` → `/opt/hermes/run_agent.py`（覆盖，免疫系统代码）
