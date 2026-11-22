# Running Talk to Figma on Linux (figma-linux)

This fork works with the unofficial [figma-linux](https://github.com/Figma-Linux/figma-linux) client, so you get the plugin bridge without the official desktop app or a Figma MCP quota.

## Changes in this fork

- **No analytics.** The plugin no longer sends usage events to Google Analytics, and `google-analytics.com` is gone from the manifest's `networkAccess`.
- **Relay on localhost only.** `src/socket.ts` binds to `127.0.0.1`, so nobody on your network can join a channel and drive your open file.
- **Fixed channel.** The plugin always joins the channel `figma`, so the agent can call `join_channel("figma")` without you reading out a random ID. Run the plugin in one file at a time.
- **`execute_code` tool.** Runs any Figma Plugin API code in the open file and returns `{result, logs}`, for anything the fixed tools can't do. A run that doesn't finish within 45 s (or `timeoutMs`) returns an error.
- **Short relay logs.** The relay logs joins, leaves and dropped sends, not full message bodies, so `journalctl --user -u figma-relay` stays readable.
- **Renamed plugin.** The dev plugin shows up as **Talk to Figma (local)**, so it isn't mixed up with the Community plugin that has the same id.

## Setup

```bash
bun install
bun run build
# MCP server for Claude Code (local build, so execute_code is included)
claude mcp add --scope user TalkToFigma -- bun ~/Documents/Projects/talk-to-figma/dist/server.js
# relay as a user service
install -Dm644 linux/figma-relay.service ~/.config/systemd/user/figma-relay.service
# edit WorkingDirectory in it if you cloned somewhere other than ~/Documents/Projects/talk-to-figma
systemctl --user daemon-reload
# optional: launcher that starts the relay, then Figma
install -Dm755 linux/figma ~/.local/bin/figma
install -Dm755 linux/figma-bridge ~/.local/bin/figma-bridge   # foreground relay, Ctrl+C to stop
```

To make your app launcher start the relay too, copy `/usr/share/applications/figma-linux.desktop` to `~/.local/share/applications/` and set `Exec=$HOME/.local/bin/figma %U`.

In Figma: **Plugins → Development → Import plugin from manifest…** → `src/cursor_mcp_plugin/manifest.json`. Then run **Talk to Figma (local)** (not the Community "Cursor MCP Plugin", which uses a random channel and analytics) in each file you open; after the first time, **Ctrl+Alt+P** re-runs it. It connects on its own.

The Plugin API can't create new Figma files. The agent works inside the file the plugin is running in, where it can add pages, frames, components and text.
