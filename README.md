# Telegram Multi-Agent Skill for OpenClaw

This skill provides a guide and example configuration for setting up a multi-agent environment in OpenClaw using Telegram. It allows you to route Telegram messages to different agents based on account ID, configure multiple Telegram bots, and enable agent-to-agent communication.

## Prerequisites

-   OpenClaw installation
-   Telegram bots created using BotFather

## Installation

1.  Copy the `telegram-multi-agent` directory to your `~/clawd/skills` directory.
2.  Copy the example configuration from `example-openclaw.json` to your `~/.openclaw/openclaw.json` file.
3.  Modify the configuration to match your environment:
    -   Replace `YOUR_MAIN_BOT_TOKEN`, `YOUR_DEV_BOT_TOKEN`, `YOUR_CONTENT_BOT_TOKEN`, and `YOUR_LAW_BOT_TOKEN` with your actual bot tokens.
    -   Update the `workspace` paths for each agent to point to your actual agent workspaces.
4.  Create the agent workspaces and SOUL.md files if they don't already exist.
5.  Restart the OpenClaw gateway: `openclaw gateway restart`.

## Configuration Details

### `agents.list`

This section defines the agents in your multi-agent environment. Each agent should have a unique `id`, a descriptive `name`, a `workspace` path, and a `model`.

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Jarvis",
        "workspace": "/path/to/main/workspace",
        "model": "google/gemini-2.0-flash"
      },
      // ... other agents
    ]
  }
}
```

### `tools.agentToAgent`

This section enables agent-to-agent communication and specifies the allowed agents. Make sure `enabled` is set to `true` and the `allow` list includes all agents that need to communicate with each other.

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": [
        "main",
        "dev",
        "content",
        "law"
      ]
    }
  }
}
```

### `channels.telegram.accounts`

This section configures the Telegram bot tokens for each agent. Each agent should have a unique `botToken`.

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "groupPolicy": "open",
      "accounts": {
        "main": {
          "botToken": "YOUR_MAIN_BOT_TOKEN"
        },
        // ... other accounts
      }
    }
  }
}
```

### `bindings`

This section routes Telegram messages to the appropriate agents based on account ID. Each binding should specify an `agentId` and a `match` object with the `channel` and `accountId`.

```json
{
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "telegram",
        "accountId": "main"
      }
    },
    // ... other bindings
  ]
}
```

## Troubleshooting

-   **Bots not responding:** Make sure the bot tokens are correct and the bots have been added to the desired groups.
-   **Agent-to-agent communication not working:** Ensure that the `agentToAgent` tool is enabled and the `allow` list includes all agents that need to communicate.

## License

MIT
