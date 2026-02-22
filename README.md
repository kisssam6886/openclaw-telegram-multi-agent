# OpenClaw Telegram Multi-Agent Skill

> 用 OpenClaw 在 Telegram 上搭建一支 AI 团队，各 Bot 各司其职，互相协作。

## 🧩 这是什么？

这个 Skill 提供了一套完整的配置方案，帮助你在 [OpenClaw](https://github.com/openclaw/openclaw) 上用多个 Telegram Bot 搭建 AI 团队：

- **Jarvis（总调度）** — 接收指令、分派任务、协调整体
- **ByteForge（开发助理）** — 代码、服务器、GitHub、构建
- **EchoSignal（内容助理）** — X/Twitter、YouTube、社区运营
- **LexVault（法务财务助理）** — 合同、合规、成本追踪

各 Agent 之间通过 `sessions_send` 工具互相通信，真正实现团队协作自动化。

---

## 🗂 文件说明

| 文件 | 说明 |
|------|------|
| `SKILL.md` | Skill 描述，供 OpenClaw 识别 |
| `example-openclaw.json` | 完整配置示例 |
| `README.md` | 本文件 |
| `templates/` | 各 Agent SOUL.md 模板 |
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
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "groups": {
        "-1001234567890": { "policy": "open" }
      },
      "accounts": {
        "main": { "botToken": "YOUR_MAIN_BOT_TOKEN", "groupPolicy": "allowlist" },
        "dev":  { "botToken": "YOUR_DEV_BOT_TOKEN",  "groupPolicy": "allowlist" },
        "content": { "botToken": "YOUR_CONTENT_BOT_TOKEN", "groupPolicy": "allowlist" },
        "law":  { "botToken": "YOUR_LAW_BOT_TOKEN",  "groupPolicy": "allowlist" }
      }
    }
  }
}
```

### 3. 创建各 Agent 的 Workspace

每个 Agent 需要自己的 workspace 目录，并在里面放 `SOUL.md`（角色定义文件）。

```bash
mkdir -p ~/.openclaw/workspace-dev
mkdir -p ~/.openclaw/workspace-content
mkdir -p ~/.openclaw/workspace-law
```

参考 `templates/` 目录里的模板文件。

### 4. 重启 Gateway

```bash
openclaw gateway restart
```

---

## ⚙️ Agent 间通信（sessions_send）

### ✅ 正确用法

由于 Telegram 机制限制，Bot 在群里互相看不到对方的消息（Bot 互盲）。正确的协作方式是用 `sessions_send`：

```
sessions_send(sessionKey="agent:dev:telegram:group:-你的群ID", message="任务内容")
```

**⚠️ 重要：`sessions_send` 只认 `sessionKey` 参数，不支持 `label`、`agentId`、`to` 等参数。**

建议在每个 Agent 的 `SOUL.md` 中直接硬编码已知的 sessionKey，避免每次都调用 `sessions_list` 查询：

```
# 各 Agent sessionKey（群）
- Dev:     agent:dev:telegram:group:-你的群ID
- Content: agent:content:telegram:group:-你的群ID
- Law:     agent:law:telegram:group:-你的群ID

# 各 Agent sessionKey（私聊）
- Dev:     agent:dev:main
- Content: agent:content:main
- Law:     agent:law:main
```

如果群 session 超时，自动改用私聊 session（`agent:dev:main` 等）。

### ❌ 常见错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `Either sessionKey or label is required` | 用了 `agentId` 或 `to` 参数 | 改用 `sessionKey` |
| Agent 在群里 @ 其他 Bot | 忘了 Bot 互盲机制 | 改用 `sessions_send` |
| Agent 用 `message` 工具发群消息失败 | `message` 工具需要显式 accountId 上下文 | 直接回复 sessions_send 消息即可，OpenClaw 会自动路由 |

### 💡 回复路由机制

Agent 收到 `sessions_send` 后，**直接回复**即可，不需要调用 `message` 工具。OpenClaw 会根据 session 的 delivery context 自动把回复路由到正确的 Telegram 频道（群或私信）。

---

## 🔒 安全配置（必读）

### 1. 禁止 Bot 被第三方拉群

默认情况下，你的 Bot 可以被任何人拉入任意群组，存在安全风险。**配置完成后务必关闭此权限**：

**方法一（推荐）：通过 BotFather 关闭**
1. 在 Telegram 找到 [@BotFather](https://t.me/BotFather)
2. 发送 `/setjoingroups`
3. 选择你的 Bot
4. 选择 **Disable**

关闭后，只有 Bot 管理员才能将 Bot 添加到群组，第三方无法随意拉群。

**方法二：通过 groupPolicy 限制**

在 `openclaw.json` 中将 `groupPolicy` 设为 `allowlist`，并在 `groups` 中只列出你的授权群 ID：

```json
"telegram": {
  "groupPolicy": "allowlist",
  "groups": {
    "-1001234567890": { "policy": "open" }
  }
}
```

这样即使 Bot 被拉入其他群，也不会响应消息。

### 2. 用户白名单配置

默认 `dmPolicy: "pairing"` 要求用户先完成配对才能与 Bot 交互，已有一定保护。如需进一步限制：

**DM 白名单**（只允许指定 Telegram 用户 ID 私聊）：
```json
"accounts": {
  "main": {
    "botToken": "YOUR_TOKEN",
    "dmPolicy": "allowlist",
    "users": ["你的Telegram用户ID"]
  }
}
```

**群组白名单**（只允许指定群响应）：
```json
"telegram": {
  "groupPolicy": "allowlist",
  "groups": {
    "-1001234567890": { "policy": "open" }
  }
}
```

> ⚠️ 你的 Telegram 用户 ID 可以通过 `/myid` 命令获取（发给你的 Jarvis Bot）。

---

## ❓ 常见问题

**Q: Bot 在群里没有回应？**
确认 Bot 已被添加到群组，`groupPolicy` 设置为 `open`（或该群 ID 在 allowlist 中），且 Bot 具有发消息权限。

**Q: sessions_send 报 "Either sessionKey or label is required"？**
确认你用的是 `sessionKey` 参数，不是 `label` 或 `agentId`。sessionKey 格式为 `agent:<agentId>:telegram:group:<群ID>`。

**Q: sessions_send 报 session 超时？**
目标 Agent 的群 session 可能过载，改用私聊 session：`agent:dev:main`。

**Q: Agent 的 session 历史积累了错误认知怎么办？**
在 Telegram 群里直接给该 Bot 发 `/reset` 或 `/new`，清空 session 历史重新开始。

**Q: 能加更多 Agent 吗？**
可以，按相同格式在 `agents.list`、`accounts`、`groups` 里各加一条即可。

---

## 📅 更新日志

### 2026-02-22
- 新增：Agent 间通信最佳实践（sessionKey 硬编码建议）
- 新增：`message` 工具 vs 直接回复的说明（直接回复即可，无需调用 message 工具）
- 新增：安全配置章节（禁止第三方拉群 + 用户白名单）
- 修复：SOUL.md 模板中错误的"必须用 message 工具发群消息"指引，改为直接回复
- 修复：sessions_send 参数说明（只认 sessionKey，不支持 label/agentId）

---

## 📄 License

MIT © [kisssam6886](https://github.com/kisssam6886)
