---
name: restivity-automation
description: 'Drive Restivity (tasks, projects, pomodoro, tags, comments) as an AI agent via its MCP server and REST API — auth, the tool catalog, and the key task/today/search/batch/recurrence/subtask workflows.'
metadata:
  version: '1.0.10'
---

# Restivity automation

Restivity is a productivity app (tasks, projects, pomodoro, tags, comments, teams).
Agents drive it two ways, both over the same authenticated, RLS-scoped data:

- **MCP** (recommended for agents): `https://restivity.ai/api/mcp` — Streamable HTTP, ~45 tools.
- **REST**: `https://restivity.ai/api/v1` — OpenAPI at `https://restivity.ai/api/v1/openapi.json`.

## Authentication (OAuth 2.1 + Dynamic Client Registration)

1. Discover: `GET /.well-known/oauth-protected-resource` and `GET /.well-known/oauth-authorization-server`.
2. Register a client: `POST /api/oauth/register` (RFC 7591 DCR), use the HTTPS URL of your Client ID Metadata Document as `client_id` (CIMD, see `/auth.md`), or use a pre-registered client.
3. Run the authorization-code + PKCE flow; exchange at `/api/oauth/token`.
4. Call `/api/mcp` (or `/api/v1`) with `Authorization: Bearer <access_token>`.

## Key task workflows

- **List / filter / sort**: `tasks_list` — the lens is `scope`: without `project_id` the default is `mine` (the tasks the user created or that are assigned to them — «my tasks»); with `project_id` the default is `all` (every task of that project the user may see, teammates' included — the board); `scope: "all"` without a project lists every task the user may see. Filter by project/status/priority/tags, `created_after`/`due_before` (ISO date OR datetime), `sort_by` + `sort_order`.
- **Full-text**: `tasks_search` (typo-tolerant). Use when the user describes a task by content, not id. READ-ONLY.
- **Recurring series are a thing apart**: `tasks_list` and `tasks_search` list the OCCURRENCES of a `materialized` series (one task per day, `parent_recurrence_id` = its master), never the master (template) itself — that is the rule, not a task to do. Pass `include_series: true` to get the masters too (to find a series and change its rule); `tasks_get` on a master id always answers. A `reminder_only` task is always listed (there the master IS the task).
- **Focus list**: `tasks_today` — timezone-aware "what should I do today" (includes overdue by default).
- **Create**: `tasks_create`. `priority`, `is_urgent`, `is_important` are INDEPENDENT (priority = list order; urgent/important = Eisenhower). `is_for_today` (focus flag) and `due_date` (deadline) are independent too.
- **Update**: `tasks_update` by id. Nullable fields: `null` clears, omit leaves unchanged. **Subtasks MERGE**: items with `id` are updated, without `id` are created, omitted-from-array are deleted, `[]` clears all.
- **Restore**: `tasks_restore` brings a deleted task back within 30 days — the person who owns it, the person who put it in the Trash, or an owner or admin of the company workspace restores it (`forbidden` with `trash_restore_forbidden` otherwise; an owner or admin who restores a task of a team they are not in gets a receipt — `restored: true`, `readable: false` — instead of the task); past the window and not purged yet it answers `restore_window_expired` — nobody can restore it, nothing is changed. Every other restore — `projects_restore`, `comments_restore`, `tags_restore`, `pomodoro_restore` (a session is its owner's alone: `not_found` for anyone else) — keeps the same 30 days and answers the same `restore_window_expired` past them; tell the person plainly, and never re-create the thing instead. A project that was the workspace's agent inbox when it went to the Trash, while another project is the inbox now, answers `agent_inbox_exists` with `agent_inbox` naming the current inbox: nothing is restored — the person turns «Use as the agent inbox» off on that project in the app, then restores; never choose another inbox for them.
- **Mass change**: `tasks_batch_update` — atomic patch over up to 50 ids, as the signed-in person. The template (master) of a `materialized` series is never written by a batch: it comes back under `skipped` (`reason: "series_template"`, a `hint`) while the other ids ARE written — tell the user which tasks were not changed and why.
- **Recurrence**: `recurrence_model="materialized"` (each occurrence is a trackable task) vs `"reminder_only"` (one task that re-prompts); set `recurring_frequency` (+ `recurring_days` for weekly) and `recurrence_start_date` ("starts on", default today — "il 15 di ogni mese da ottobre" = monthly + start 2026-10-15). The deadline is only a deadline: a reminder is due on the occurrence it is on (server-owned — never send `due_date` for it; completing it moves it to the next one, `tasks_recurrence_skip` moves it without a completion); a materialized master has no `due_date` (each child has its own — a `due_date` or `is_for_today` on it answers `series_template`). Change a materialized series' rule with `tasks_recurrence_update_rule` (it keeps, moves and creates the occurrences): `tasks_update` with a rule field on its master answers `conflict_recurring_master`; its title, description and priority through `tasks_update` change the master only.

Other families: `projects_*` (incl. `projects_agent_inbox` — the workspace's agent inbox, where findings are filed; it is created only on the person's request: the first time it answers `agent_inbox_missing` with a `proposal` and creates nothing — ask the person whether to create the inbox (name, prefix) or use an existing project, then call again with `project_id` or with `confirm_create: true` and the name and prefix they chose; `AGF` is a free key — and `projects_statuses_list/create/update/delete/reorder/customize/follow/restore_base` — a project's board stages AND its inheritance switches), `workspaces_statuses_list/create/update/delete/reorder/restore_base` and `teams_statuses_list/create/update/delete/reorder/customize/follow/restore_base` (RES-120: the palette at the three levels — workspace default, team override, project override; a project/team without its own FOLLOWS the level above and refuses project/team writes with `inherited_palette` until customised; the workspace/team levels need a paid plan, `upgrade_required`; `*_statuses_restore_base` runs in two calls: the first, without `confirmation_token`, changes nothing and answers the recap and a token — read the recap to the person and, only after they confirm, call again with that `confirmation_token` (it lives 10 minutes; `confirmation_stale` means the list changed: read them the new recap); a project has no such restore — `projects_statuses_restore_base` answers `project_restores_by_following`, its way back is `projects_statuses_follow`), `teams_unarchive` (reopens an archived team: an admin of the team, or the owner or an admin of its workspace — `forbidden` otherwise; a team in the Trash answers `team_in_trash` (or `workspace_in_trash`), a team that is open `team_not_archived`; an archived team is never joined, not even by its admins — it is reopened first), `pomodoro_*`, `tags_*`, `comments_*`, `profile_*`; composites `focus_start` / `focus_finish` (see the `restivity-focus-flow` skill); `skills_get` returns the current text of these skills (`{name}` → the body and its version; no `name` → the list) — call it before writing a task (`restivity-task-writing`) or a comment (`restivity-task-updates`). Every tool documents its scope, "use when", and common error codes (not_found, forbidden, validation_error, conflict_recurring_master, overlapping_active_session, …).

## Names

**Names that look alike — ask, never guess**: when the user names a client, project, team or tag that resembles two or more existing ones (the same words in another case or spacing, one a typo away from another, one name inside another — «Sito» and «Sito web»), or the name is otherwise ambiguous, ask which one they mean before acting, list the candidates, and act only on their answer — never pick one yourself, and never create a new one because no match was exact. Names are unique where a duplicate would confuse (a team's projects, a workspace's teams and team-less projects, a person's tags, a workspace's clients — compared without case and surrounding spaces): a create or rename onto a taken name is refused with `name_taken`; tell the user and offer the existing one instead of retrying with a variant of the name.

## Discovery surfaces

- MCP Server Card: `/.well-known/mcp/server-card.json`
- API catalog: `/.well-known/api-catalog`
- This index: `/.well-known/agent-skills/index.json`
