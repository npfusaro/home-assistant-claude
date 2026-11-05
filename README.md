# Home Assistant + Claude Code

Lets [Claude Code](https://claude.com/claude-code) manage a Home Assistant instance
through [ha-mcp](https://github.com/homeassistant-ai/ha-mcp), with a guided setup that
takes about ten minutes. Nothing is installed on Home Assistant itself. Works on macOS,
Linux and Windows.

## Get started

You need Claude Code, Python 3.13 or newer (or [uv](https://docs.astral.sh/uv/)), and an
administrator account on your Home Assistant.

Clone this repository and start Claude with the setup prompt:

```
git clone https://github.com/npfusaro/home-assistant-claude.git
cd home-assistant-claude
claude "follow SETUP.md"
```

Claude installs and checks everything. You do two things yourself: create a Home
Assistant access token, and paste it into one command in your own terminal. The token
never goes into the chat.

Afterwards, just run `claude` in the folder and ask for what you want. Two other
starting prompts are useful later:

```
claude "follow SETUP.md from phase 6"    # check everything still works, e.g. after an HA update
claude "follow SETUP.md phase 9"         # update the MCP server
```

## What's here

| File | Purpose |
|---|---|
| `SETUP.md` | The guided setup and health check |
| `CLAUDE.md` | The rules Claude follows in this folder |
| `.claude/settings.json` | Pre-approves the read-only tools; anything that changes Home Assistant asks first |
