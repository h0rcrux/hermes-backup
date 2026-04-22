---
name: docker-mutual-repair
description: Set up bidirectional mutual repair between two Docker containers — each can diagnose, repair, and restart the other via shared Docker Socket.
tags: [docker, container, repair, monitoring, mutual-watch]
---

# Docker Mutual Repair — 双向互保

两个 Docker 容器互相监控和修复的方案。核心：共享 Docker Socket + Doctor 脚本 + 守护进程。

## 关键原则

**路径写法**：脚本里的路径必须是你有写权限的地方。不要写"理想路径"（如 `/app/`）再靠挂载弥补，直接写你能写的地方（如 `/opt/{容器名}/`）。

**对称命名**：
```
/opt/{container-a}/repair/     → container-a 修 container-b 的工具
/opt/{container-a}/logs/       → container-a 的日志
/opt/{container-b}/repair/     → container-b 修 container-a 的工具
/opt/{container-b}/logs/       → container-b 的日志
```

## 文件结构

每个容器需要两个脚本：

| 脚本 | 作用 |
|------|------|
| `{peer}-doctor.sh` | 诊断+修复对方容器（status/logs/exec/diagnose/fix/fix-mem/fix-restart） |
| `mutual-watch-{self}.sh` | 后台守护进程，每60秒检查对方健康状态 |

## 必需配置

1. **Docker Socket 挂载** — 两个容器都要挂载：
   - 主机路径：`/var/run/docker.sock`
   - 容器路径：`/var/run/docker.sock`

2. **容器内安装 docker-cli** — Debian 系：`apt-get install -y docker.io`

## Doctor 脚本核心功能

```bash
{peer}-doctor status      # 查看对方状态
{peer}-doctor logs 100    # 查看最近100行日志
{peer}-doctor diagnose    # 全面诊断（容器状态/资源/错误日志/进程/网络）
{peer}-doctor fix         # 自动修复
{peer}-doctor fix-mem     # 修复内存问题
{peer}-doctor fix-restart # 重启容器
{peer}-doctor exec "cmd"  # 在对方容器执行命令
{peer}-doctor shell       # 进入对方容器
```

## Watch 守护进程核心逻辑

```
while true:
  1. 检查对方是否存活
  2. 获取健康状态 (HEALTHY / DEGRADED / DEAD_OOM / UNHEALTHY)
  3. 根据状态自动调用 doctor 修复
  4. sleep 60秒
```

健康判断标准：
- 内存 > 90% → DEAD_OOM → fix-mem
- 内存 > 80% → DEGRADED → diagnose
- 5分钟内错误 > 20 → DEGRADED → diagnose + fix
- 容器未运行 → 尝试 restart/start

## 路径修改 Checklist

创建脚本后，全局替换以下路径：

| 原始路径 | 改为 |
|----------|------|
| `/app/logs/` | `/opt/{容器名}/logs/` |
| `/app/repair/` | `/opt/{容器名}/repair/` |

脚本里所有引用都要改，包括：
- `LOG_FILE` 变量
- `mkdir -p` 命令
- 调用 doctor 脚本的路径

## 常见坑

- **权限问题**：确保脚本有执行权限 `chmod +x *.sh`
- **容器名不匹配**：脚本里的 `PEER_CONTAINER` 和 `SELF_CONTAINER` 要和实际容器名一致
- **路径写错**：最常见错误，创建脚本时直接写目标路径，不要事后靠挂载补
- **Docker CLI 缺失**：容器内没装 docker 命令，doctor 会尝试自动安装但可能失败
