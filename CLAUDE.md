# Home Assistant via ha-mcp

This folder is for managing a Home Assistant instance through the `ha` MCP server
(ha-mcp). The live instance is the only source of truth; nothing is copied locally.

## Rules

- Reach Home Assistant only through the `mcp__ha__*` tools. Never ssh, rsync, scp or
  curl to it, and never ask for or print the access token.
- Before creating or changing an automation, script, scene, helper or dashboard, call
  `ha_get_skill_guide` and read the reference file its table names for that task.
- Read before you write: fetch the object with `ha_config_get_*` and pass its
  `config_hash` on updates, so an edit made in the HA UI meanwhile is detected.
- Never guess entity IDs. Confirm them with `ha_search` or `ha_get_entity`.
- Keep results small: pass `fields` / `result_fields`; use `ha_get_overview` only with
  `detail_level="minimal"`.
- Every write is backed up first by ha-mcp (per-edit backups under `~/.ha-mcp/backups`).
  Undo with `ha_manage_backup(scope="edits", action="list" | "diff" | "restore")`.
- Some things exist only as files on the HA box (`configuration.yaml`, ESPHome YAML,
  packages, themes) and have no API. For those, write out the exact edit (file, full
  block, why), ask the user to apply it in the File editor add-on, then run
  `ha_get_system_health(include="config_check")` and the matching reload.
- Ask before removing anything, restarting Home Assistant, installing or updating
  add-ons or store items, or restoring a backup.
- If a tool result says a newer ha-mcp release is available, mention it to the user.
  Updating is phase 9 of `SETUP.md`; never do it unasked.

## Which tool

| Task | Tool |
|---|---|
| Find entities, and what references one | `ha_search` |
| State and registry details; rename, area, icon | `ha_get_state`, `ha_get_entity`, `ha_set_entity` |
| Automations, scripts, scenes | `ha_config_get_*`, `ha_config_set_*`, `ha_config_remove_*` |
| Why didn't it fire | `ha_get_automation_traces`, then `ha_get_logs(source="system")` |
| Test a template or condition | `ha_eval_template` |
| Helpers | `ha_config_list_helpers`, `ha_config_set_helper` |
| Dashboards and card resources | `ha_config_get_dashboard`, `ha_config_set_dashboard`, `ha_config_*_dashboard_resource` |
| Install cards, themes, integrations | `ha_manage_hacs` |
| Config check, logs, add-ons | `ha_get_system_health`, `ha_get_logs`, `ha_get_app` |
| Reload / restart | `ha_call_service(domain=..., service="reload")`, `ha_restart` |

## House notes

(Add facts about this particular home here as they come up: naming conventions,
rooms, things that must never be automated, and so on.)

Setup and health check: `SETUP.md`.
