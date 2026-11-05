# Home Assistant + Claude Code setup

**For the human:** open a terminal in this folder and run `claude "follow SETUP.md"`.
Claude installs and checks everything; you do the two steps that involve your Home
Assistant token. It takes about ten minutes and works on macOS, Linux and Windows.

You need: Claude Code, network access to your Home Assistant, and an administrator
account on it. Nothing gets installed on the Home Assistant box itself.

To re-check things later (for example after a Home Assistant update), run
`claude "follow SETUP.md from phase 6"`. To update ha-mcp itself, run
`claude "follow SETUP.md phase 9"`.

---

**For Claude:** you are guiding the user through installing [ha-mcp](https://github.com/homeassistant-ai/ha-mcp),
the Home Assistant MCP server, and verifying it. Work through the phases in order.
Before each phase say in one line what you are about to do; after it, say what
happened. Never ask the user to paste a token or any secret into this conversation,
and never print one. If a step fails, stop, show the error, and sort it out with the
user before moving on.

## Phase 1 — Preflight (Claude)

1. Detect the operating system and shell, and note them: everything below has
   macOS/Linux and Windows variants.
2. Check Claude Code: `claude --version`.
3. Find Python 3.13 or 3.14 (ha-mcp needs ≥ 3.13, < 3.15). Try, in order:
   `python3.13 --version`, `python3 --version`, `py -3.13 --version` (Windows),
   `uv --version`. If none fits, give the user the install line for their OS and wait:

   | OS | Install |
   |---|---|
   | Debian / Ubuntu | `sudo apt install python3 python3-venv` (Debian 13 ships 3.13) |
   | macOS | `brew install python@3.13` |
   | Windows | `winget install Python.Python.3.13` |
   | Any, alternative | uv: `curl -LsSf https://astral.sh/uv/install.sh \| sh` (macOS/Linux) or `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 \| iex"` (Windows) |

4. Run `claude mcp list`. If it shows another Home Assistant server (names like
   `hass-mcp`, `home-assistant`, `homeassistant`), explain that two servers would give
   duplicate tools, and ask the user to remove the old one in their terminal:
   `claude mcp remove <name> -s user` (or `-s local` if it was added for this folder).
   Wait until they confirm.

## Phase 2 — Project files (Claude)

These three files ship with the repository. Confirm each one is present and tell the
user in one sentence what it is for:

- `CLAUDE.md` — the rules Claude follows when working on this Home Assistant.
- `.claude/settings.json` — pre-approves the read-only `ha` tools so questions never
  trigger a prompt. Anything that changes Home Assistant asks the first time it is
  used, and that prompt offers "don't ask again" per tool. It also blocks `ssh`,
  `rsync` and `scp` inside this folder.
- `.gitignore` — keeps the virtual environment and local settings out of git.

If any is missing, the clone is incomplete: ask the user to clone the repository again
rather than recreating files by hand.

## Phase 3 — Install ha-mcp (Claude)

Create a virtual environment in this folder and install the current release (this
guide was written against 9.1; anything newer is fine, phase 6 verifies it).

macOS / Linux:

```
python3 -m venv .venv
.venv/bin/pip install --upgrade pip
.venv/bin/pip install "ha-mcp>=9.1"
.venv/bin/ha-mcp --help
```

Windows (PowerShell):

```
py -3.13 -m venv .venv
.venv\Scripts\pip install --upgrade pip
.venv\Scripts\pip install "ha-mcp>=9.1"
.venv\Scripts\ha-mcp.exe --help
```

With uv instead of a system Python (any OS): `uv venv --python 3.13 .venv` then
`uv pip install "ha-mcp>=9.1"`.

The last command is expected to print a **"Configuration Error / Missing required
environment variables"** banner. That is correct: it proves the install works and the
credentials simply are not set yet. Write down the absolute path of the executable
(`<this folder>/.venv/bin/ha-mcp`, or `<this folder>\.venv\Scripts\ha-mcp.exe` on
Windows). Phase 4 needs it.

## Phase 4 — Connect it to Home Assistant (the user, in their own terminal)

This is the one step the user does by hand, so the token never passes through this
conversation. Give them these instructions, filled in with the real executable path
from phase 3, and then wait.

1. **Create a token.** In Home Assistant, click your name at the bottom-left, open the
   **Security** tab, scroll to **Long-lived access tokens**, click **Create token**, name
   it `claude-ha-mcp`, and copy it. It is shown only once; if you lose it, create another.
2. **Find your Home Assistant address.** It is the address in your browser, for example
   `http://192.168.1.50:8123` or `http://homeassistant.local:8123`. Use the local one
   when this computer is on the same network.
3. **Open a second terminal window** (not this Claude conversation), go to this folder,
   and run the following, replacing the two placeholders. Do not run it inside Claude
   Code with the `!` prefix: that output lands in the conversation, token included.

   It is one command split over five lines so it survives copy and paste; paste all
   five lines together. Claude: print it exactly in this form, with the real path
   filled in, inside a code block. Never join it onto one line and never break a line
   anywhere else, because a stray line break makes the shell run half a command.

   macOS / Linux:

   ```
   claude mcp add ha \
     -e HOMEASSISTANT_URL=http://YOUR-HA-ADDRESS:8123 \
     -e HOMEASSISTANT_TOKEN=PASTE-YOUR-TOKEN-HERE \
     -e ENABLE_MANDATORY_BPS=false \
     -- /ABSOLUTE/PATH/TO/.venv/bin/ha-mcp
   ```

   Windows (PowerShell):

   ```
   claude mcp add ha `
     -e HOMEASSISTANT_URL=http://YOUR-HA-ADDRESS:8123 `
     -e HOMEASSISTANT_TOKEN=PASTE-YOUR-TOKEN-HERE `
     -e ENABLE_MANDATORY_BPS=false `
     -- C:\ABSOLUTE\PATH\TO\.venv\Scripts\ha-mcp.exe
   ```

   Expected reply: `Added stdio MCP server ha ...`. The token is now stored in Claude
   Code's own configuration for this folder, the same place every MCP server keeps its
   settings. Tip: on macOS/Linux, start the first line with a space so the command
   stays out of your shell history.
4. Come back here and say **"done"**.

When the user says done, run `claude mcp list` and confirm a line starting with `ha:`
appears (do not print anything else from that output). If it is missing, the command
was probably run in a different folder; ask them to run it again from this one.

## Phase 5 — Restart Claude Code (the user)

MCP servers load when Claude Code starts. Ask the user to:

1. Type `/exit`.
2. In the same folder, run: `claude "follow SETUP.md from phase 6"`

## Phase 6 — Test the connection (Claude, using the `ha` tools)

First confirm the `mcp__ha__*` tools are available in this session. If they are not,
see Troubleshooting at the end before going further.

Run each check below and fill in the table. Use only the parameters shown. If a check
fails, stop and troubleshoot with the user; do not skip ahead.

| # | Check | Call | Expected |
|---|---|---|---|
| 1 | Skill guide | `ha_get_skill_guide()` | The best-practices SKILL.md text |
| 2 | Entity state | `ha_get_state(entity_id="sun.sun", fields=["state"])` | `above_horizon` or `below_horizon` |
| 3 | Search | `ha_search(query="light", limit=3, result_fields=["entity_id", "friendly_name"])` | A few entities |
| 4 | Find an automation | `ha_eval_template(template="{{ states.automation \| map(attribute='entity_id') \| list \| first }}")` | An `automation.*` id (if the instance has none, mark 5–6 n/a) |
| 5 | Automation config | `ha_config_get_automation(entity_id=<id from 4>)` | Its config and a `config_hash` |
| 6 | Traces | `ha_get_automation_traces(automation_id=<id from 4>, limit=1)` | success (zero traces is fine) |
| 7 | Config check | `ha_get_system_health(include="config_check")` | `"result": "valid"` |
| 8 | Logs | `ha_get_logs(source="system", limit=3, compact=true)` | success (empty is fine) |
| 9 | Dashboards | `ha_config_get_dashboard(list_only=true)` | The dashboard list |
| 10 | Helpers | `ha_config_list_helpers(helper_type="input_boolean", limit=3)` | success (zero is fine) |
| 11 | Add-ons | `ha_get_app(source="installed")` | Add-on list on HA OS / Supervised; on Container or Core an error about the Supervisor is normal, mark n/a |
| 12 | Create (disabled) | `ha_config_set_automation(config={"alias": "[setup test] delete me", "description": "created by SETUP.md, safe to delete", "triggers": [{"trigger": "state", "entity_id": "sun.sun"}], "actions": [{"action": "persistent_notification.create", "data": {"title": "setup test", "message": "this should never fire"}}]}, enabled=false)` | `operation: created`; note its `entity_id` and `config_hash` |
| 13 | Update | `ha_config_set_automation(entity_id=<from 12>, config_hash=<from 12>, python_transform="config['description'] = 'updated by SETUP.md'")` | `operation: updated` |
| 14 | Per-edit backup | `ha_manage_backup(scope="edits", action="list", domain="automation")` (this tool is not pre-approved; the user approves the prompt) | At least one backup for the test automation |
| 15 | Delete | `ha_config_remove_automation(entity_id=<from 12>)`, then `ha_config_get_automation(entity_id=<from 12>)` | `deleted`, then "not found" |

Report the table with a Result column (pass / fail / n/a) and one line per failure.

## Phase 7 — Backups (Claude)

ha-mcp backs up each object before it changes it (check 14). The disaster-recovery copy
is Home Assistant's own scheduled backup. Run
`ha_manage_backup(scope="snapshot", action="list")` and look for a backup made in the
last two days. If there is none, tell the user:

> Home Assistant is not making automatic backups. Open **Settings → System → Backups**,
> click **Configure automatic backups**, pick daily, keep at least three copies, and
> choose a location. This is the safety net for anything the per-edit backups do not
> cover.

## Phase 8 — Done (Claude)

Tell the user, in a few lines:

- What is installed and where: ha-mcp in `.venv/`, registered with Claude Code for this
  folder as `ha`; per-edit backups in `~/.ha-mcp/backups`; the rules Claude follows in
  `CLAUDE.md`; read-only tools pre-approved in `.claude/settings.json`, so the first
  time a write tool is used the user gets a prompt and can choose "don't ask again".
- How to use it: just ask. Examples: "what's on in the kitchen", "why didn't the porch
  light automation fire last night", "create an automation that turns the hallway light
  on at sunset", "add a card for the garage door to the Overview dashboard".
- How to check it later: `claude "follow SETUP.md from phase 6"`.
- How to update ha-mcp: `claude "follow SETUP.md phase 9"`. ha-mcp checks for a newer release
  each time it starts and reports it inside `ha_get_system_health` and
  `ha_get_overview` results; Claude mentions it when it sees it but never updates on
  its own.

Optional, if they want it later: Home Assistant's own tracked changes
("Settings → Activity"), and a `## House notes` section in `CLAUDE.md` for facts about
their home.

## Phase 9 — Update ha-mcp (Claude, only when the user asks)

Updating never needs re-registration: Claude Code points at the executable in `.venv/`,
and the upgrade replaces it in place. The bundled best-practices skill updates with it.

1. Record the current version: `.venv/bin/pip show ha-mcp` (Windows:
   `.venv\Scripts\pip show ha-mcp`). Tell the user the version.
2. Upgrade: `.venv/bin/pip install --upgrade ha-mcp` (Windows:
   `.venv\Scripts\pip install --upgrade ha-mcp`). Tell the user the new version.
3. The running server is the old one until Claude Code restarts. Ask the user to
   `/exit` and then run `claude "follow SETUP.md from phase 6"`.
4. If phase 6 then fails on something that passed before, roll back with
   `.venv/bin/pip install ha-mcp==<old version>` and restart again, and tell the user
   which release broke what so they can report it upstream.

## Troubleshooting

- **No `mcp__ha__*` tools in the session.** Run `claude mcp list` in the terminal. If
  `ha` is missing, redo phase 4 from this folder. If it shows `✘ Failed`, run the
  executable by hand with the two variables set in that terminal (the user does this,
  not Claude) to see the real error. A wrong URL or token is the usual cause: run
  `claude mcp remove ha` and redo phase 4.
- **The terminal says `-e: command not found` or `ENABLE_MANDATORY_BPS=false: command
  not found` after pasting.** The command was pasted with a line break in the wrong
  place. Paste the five-line form from phase 4 again, all lines at once.
- **Checks time out or fail with 401.** The token was revoked or mistyped. Create a new
  one and redo phase 4.
- **Check 11 errors on an HA OS install.** The token's user must be an administrator.
- **Check 14 shows no backup.** Run `ha_manage_backup(scope="edits", action="list")`
  without a domain; if still empty, the server was started without
  `ENABLE_AUTO_BACKUP` (it is on by default). Redo phase 4 adding
  `-e ENABLE_AUTO_BACKUP=true`.
- **Slow first call.** The server takes 5–10 seconds to start; later calls are fast.
