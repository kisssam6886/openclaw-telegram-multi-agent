## Telegram Multi-Agent Skill

This skill provides a guide and example configuration for setting up a multi-agent environment in OpenClaw using Telegram.

### Features

- Route Telegram messages to different agents based on account ID.
- Configure multiple Telegram bots, each with its own persona and workspace.
- Enable agent-to-agent communication for seamless collaboration.

### Usage

1.  Copy the example configuration from `example-openclaw.json` to your `~/.openclaw/openclaw.json` file.
2.  Modify the configuration to match your environment, including bot tokens, account IDs, and agent workspaces.
3.  Create the necessary agent workspaces and SOUL.md files.
4.  Restart the OpenClaw gateway.

### Configuration

The following settings need to be configured in `~/.openclaw/openclaw.json`:

-   `agents.list`: Define the agents, including their IDs, names, workspaces, and models.
-   `tools.agentToAgent`: Enable agent-to-agent communication and specify the allowed agents.
-   `channels.telegram.accounts`: Configure the Telegram bot tokens and account IDs.
-   `bindings`: Route Telegram messages to the appropriate agents based on account ID.

### Example Configuration

See `example-openclaw.json` for a complete example configuration.

### Notes

-   Make sure each Telegram bot has been created using BotFather and has been added to the desired groups.
-   Ensure that the necessary permissions have been granted to each bot.
-   Agent-to-agent communication requires the `sessions_send` tool and the `agentToAgent` configuration to be enabled.
