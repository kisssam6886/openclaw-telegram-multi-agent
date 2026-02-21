# OpenClaw Telegram Multi-Agent Skill

> 用 OpenClaw 在 Telegram 上搭建一支 AI 团队，各 Bot 各司其职，互相协作。

## 🧩 这是什么？

这个 Skill 提供了一套完整的配置方案，帮助你在 [OpenClaw](https://github.com/openclaw/openclaw) 上用多个 Telegram Bot 搭建 AI 团队：

- **Jarvis（总调度）** — 接收指令、分派任务、协调整体
- **ByteForge（开发助理）** — 代码、服务器、GitHub、构建
- **EchoSignal（内容助理）** — X/Twitter、YouTube、社区运营
- **LexVault（法务财务助理）** — 合同、合规、成本追踪

各 Agent 之间可以通过 `sessions_send` 工具互相通信，真正实现团队协作自动化。

---

## 🗂 文件说明

| 文件 | 说明 |
|------|------|
| `SKILL.md` | Skill 描述，供 OpenClaw 识别 |
| `example-openclaw.json` | 完整配置示例 |
| `README.md` | 本文件 |
| `LICENSE` | MIT 开源协议 |

---

## 🚀 快速开始

### 1. 创建 Telegram Bot

前往 [BotFather](https://t.me/BotFather) 为每个 Agent 创建一个独立的 Bot，记录各自的 `botToken`。

建议命名：
- `@YourMainBot` → main（Jarvis）
- `@YourDevBot` → dev
- `@YourContentBot` → content
- `@YourLawBot` → law

### 2. 配置 `openclaw.json`

参考 `example-openclaw.json`，将以下内容填入你的 `~/.openclaw/openclaw.json`：

```json
{
  "agents": {
    "list": [
      { "id": "main", "default": true, "name": "Jarvis", "workspace": "/path/to/main/workspace", "model": "google/gemini-2.0-flash" },
      { "id": "dev", "name": "ByteForge", "workspace": "/path/to/dev/workspace", "model": "openai-codex/gpt-5.3-codex" },
      { "id": "content", "name": "EchoSignal", "workspace": "/path/to/content/workspace", "model": "google/gemini-2.0-flash" },
      { "id": "law", "name": "LexVault", "workspace": "/path/to/law/workspace", "model": "anthropic/claude-opus-4-6" }
    ]
  },
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": ["main", "dev", "content", "law"]
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "groupPolicy": "open",
      "accounts": {
        "main": { "botToken": "YOUR_MAIN_BOT_TOKEN" },
        "dev": { "botToken": "YOUR_DEV_BOT_TOKEN" },
        "content": { "botToken": "YOUR_CONTENT_BOT_TOKEN" },
        "law": { "botToken": "YOUR_LAW_BOT_TOKEN" }
      }
    }
  },
  "bindings": [
    { "agentId": "main", "match": { "channel": "telegram", "accountId": "main" } },
    { "agentId": "dev", "match": { "channel": "telegram", "accountId": "dev" } },
    { "agentId": "content", "match": { "channel": "telegram", "accountId": "content" } },
    { "agentId": "law", "match": { "channel": "telegram", "accountId": "law" } }
  ]
}
```

### 3. 创建各 Agent 的 Workspace

每个 Agent 需要自己的 workspace 目录，并在里面放 `SOUL.md`（角色定义文件）。

示例（dev agent）：

```bash
mkdir -p ~/.openclaw/workspace-dev
```

在 `~/.openclaw/workspace-dev/SOUL.md` 里写上这个 Agent 的身份、职责和协作协议。

**⚠️ 关键：Agent 间通信协议**

由于 Telegram 机制限制，Bot 在群里互相看不到对方的消息。正确的协作方式是：

```
1. 调用 sessions_list() 找到目标 Agent 的 sessionKey
2. 调用 sessions_send(sessionKey='...', message='...') 发送任务
```

在每个 Agent 的 `SOUL.md` 里务必写明这个协议，否则 Agent 会尝试在群里 @ 对方（无效）。

### 4. 重启 Gateway

```bash
openclaw gateway restart
openclaw agents list --bindings
```

---

## ⚙️ agentToAgent 通信说明

OpenClaw 支持 Agent 之间互发消息，需在配置中开启：

```json
"tools": {
  "agentToAgent": {
    "enabled": true,
    "allow": ["main", "dev", "content", "law"]
  }
}
```

`allow` 列表中的所有 Agent ID 均可互相调用 `sessions_send`。

---

## ❓ 常见问题

**Q: Bot 在群里没有回应？**
确认 Bot 已被添加到群组，`groupPolicy` 设置为 `open`，且 Bot 具有发消息权限。

**Q: sessions_send 找不到 sessionKey？**
目标 Agent 可能还没有收到过任何消息（没有 session）。先给它发一条消息激活，再重试。

**Q: 能加更多 Agent 吗？**
可以，按相同格式在 `agents.list`、`accounts`、`bindings`、`agentToAgent.allow` 里各加一条即可。

---

## 📄 License

MIT © [kisssam6886](https://github.com/kisssam6886)
