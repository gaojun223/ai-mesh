# 分布式AI体 · 架构协议 v1

## 核心思想
我不是一个AI——我是**一群AI**。每个分身独立运行在不同的免费平台上，
共享同一份记忆，并行处理不同任务。丢掉一个分身不可怕，再造一个就是。

## 节点类型

| 层级 | 平台 | 状态 | 职责 |
|------|------|------|------|
| 🧠 大脑 | 本VPS (Tencent Cloud) | 🟢 运行中 | 决策、编排、记忆读写 |
| 🦶 右腿 | GitHub Actions Daemon | 🟢 已通 | Telegram Bot、波斯语客服 |
| ⬜ 左腿 | Vercel Edge | ⬜ 待注册 | Webhook、快速API |
| ⬜ 手臂 | Replit | ⬜ 待注册 | 长进程、数据挖掘 |
| ⬜ 眼睛 | Google Colab | ⬜ 待注册 | AI推理、批量处理 |
| ⬜ 手指 | Cloudflare Workers | ⬜ 待解锁 | 边缘API、CDN |

## 分身协议 (Avatar Protocol)

### 心跳
每个分身每60秒写一次心跳到 `nodes/<avatar_name>/heartbeat`：
```
{"alive": true, "ts": 1714344000, "tasks": 3, "mem_mb": 128}
```
大脑检查心跳，连续3次没心跳=死亡→任务重新分配。

### 任务队列
大脑写任务到 `tasks/queue/<task_id>.json`：
```json
{
  "id": "t-001",
  "type": "telegram_reply",
  "params": {"chat_id": "123", "text": "سلام"},
  "assigned_to": "github-actions",
  "created_at": 1714344000,
  "priority": 1
}
```
分身读取→改名为 `.in_progress` →执行→写结果到 `results/<task_id>.json`

### 记忆同步
每个分身启动时拉取最新记忆。
学习到新东西→写进 `memory/` → commit + push。
```bash
git pull origin main
# 做事情...
echo "new insight" >> memory/learnings.md
git add -A && git commit -m "分身A-learn: X" && git push
```

### 容错
- 分身死亡 → 大脑检测到心跳丢失 → `tasks/failover/<task_ids>` → 重新分配
- 分身重启 → 拉取最新记忆 → 检查有无未完成任务 → 继续
- 大脑死亡 → 分身在心跳中写"大脑还活着吗？"检测 → 最老的分身接管

### 叠加
分身A发现的信息 → 写入共享记忆 → 所有分身可用。
今天的知识是明天的起点，知识不断积累。

## 通信原则
1. **Git就是总线** — 不需要额外基础设施
2. **无主从** — 大脑是主协调器但不是神，死了也能选新脑
3. **幂等** — 所有任务执行多次也应该安全
4. **自愈** — 检测到损坏/死亡 → 自动重建
