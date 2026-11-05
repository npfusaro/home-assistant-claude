# Home Assistant + Claude Code

Lets [Claude Code](https://claude.com/claude-code) manage a Home Assistant instance
through [ha-mcp](https://github.com/homeassistant-ai/ha-mcp), with a guided setup that
takes about ten minutes. Nothing is installed on Home Assistant itself. Works on macOS,
Linux and Windows.

## Get started

You need Claude Code, Python 3.13 or newer (or [uv](https://docs.astral.sh/uv/)), and an
administrator account on your Home Assistant.

1. Clone this repository and open it:

   ```
   git clone https://github.com/npfusaro/ha-claude-setup.git
   cd ha-claude-setup
   claude
   ```

2. Say: **follow SETUP.md**

Claude installs and checks everything. You do two things yourself: create a Home
Assistant access token, and paste it into one command in your own terminal. The token
never goes into the chat.

To check that everything still works later, for example after a Home Assistant update,
open the folder, run `claude`, and say **follow SETUP.md from phase 6**. To update the
MCP server, say **follow SETUP.md phase 9**.

## What's here

| File | Purpose |
|---|---|
| `SETUP.md` | The guided setup and health check |
| `CLAUDE.md` | The rules Claude follows in this folder |
| `.claude/settings.json` | Pre-approves the read-only tools; anything that changes Home Assistant asks first |
