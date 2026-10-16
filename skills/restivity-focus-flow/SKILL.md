---
name: restivity-focus-flow
description: "Use when an agent is asked to actually work on a Restivity task (implement, fix, write, research it) rather than only read or edit it, and the effort must show up in Restivity as task status, pomodoro work sessions and a closing comment. Triggers: 'work RES-58', 'lavora il task', 'take this task', 'track my time on it', an overlapping_active_session error."
metadata:
  version: '1.0.2'
---

# Restivity focus flow

Work a task the way a disciplined Restivity user does: **in progress → work session(s) → closing comment → completed**. Every minute of effort lands in a pomodoro work session (that is what Earnings, streaks and reports read); every outcome lands on the task. Two composite tools do the two halves in the right order: `focus_start` and `focus_finish`. The primitives (`tasks_update`, `pomodoro_create`, `pomodoro_update`, `comments_create`) remain for the clock rule, pauses, and servers that predate the composites.

## Before you start

1. Read the task once: `tasks_get {id: "RES-58"}` — the code works as `id` (an older server wants the UUID: `tasks_search {q: "RES-58"}`, keep the exact-code hit). Description, subtasks, current status. Already `completed` / `archived`? Ask before reopening. `ambiguous_code` (the code exists in two of the user's workspaces) → pick the candidate UUID the user means.
2. A running session on ANOTHER task is the user's own timer: never close it silently. `focus_start` refuses with `overlapping_active_session` in that case; `pomodoro_list {status: ["active", "paused"]}` names the task it runs on, so you can ask. (If the server answers `expected array, received string`, your client serialised the array as a JSON string — newer servers accept that spelling; on an older one call once per status.)

## Start

One call: `focus_start {task: "RES-58", plan: "<one-line plan>"}`. The server puts the task in progress (the project's custom statuses are honoured), opens a work session of the user's configured length with `notes: "<code> — <plan>"`, or reuses the live session already on this task (`session_reused: true`; a paused one is resumed). Keep the returned `session.id`, `start_time`, `end_time`. Do NOT call `profile_get` for a duration.

- `overlapping_active_session` → the user's timer runs on another task. Ask; then close that session with `pomodoro_update` (`completed` if its work ended, `interrupted` otherwise) and call `focus_start` again.
- Older server without `focus_start` (not in the tool list) → the primitives, in this order: `tasks_update {id, status: "in_progress"}` (or the `project_status_id` whose `category` is `in_progress`, from `projects_statuses_list {id: project_id}`), then `pomodoro_create {task_id: <uuid>, project_id, type: "work", notes: "<code> — <plan>"}`. Omit `duration`.

## While working — the clock rule

You have no timer. At every checkpoint (before any Restivity call, after each chunk of work) compare now (`date -u`, else the last server timestamp you saw) with the session's `end_time`.

- now ≥ `end_time` → close the block: `pomodoro_update {id, status: "completed", end_time: <now, ISO with Z>, elapsed_time: <now − start_time, SECONDS>, completion_notes: "block N: <done>"}`, then `focus_start {task, plan: "block N+1: <next>"}` opens the next one (the task is already in progress; no live session → a new one). Work sessions only — no `short-break` / `long-break` you did not take.
- The task stays `in_progress` across blocks.

## Finish

1. Verify the work (tests run, artefact read back) BEFORE touching Restivity.
2. Subtasks you actually did: `tasks_update {id, subtasks: [...]}` with `completed: true` on those — MERGE semantics, so send every subtask you keep, with its `id`.
3. One call: `focus_finish {task, summary: "<what changed (files, commit / PR) · tests run and their result · decisions · open points · findings filed · time tracked (N sessions, M min)>"}`. Shape the summary as the `restivity-task-updates` skill says — a bold lead, one short list, a closing `Next:` line — and, where the server accepts it, send the closing comment as a TipTap `content` document rather than a plain string. The server closes the live session on this task (`completed`, real `end_time` / `elapsed_time` — finishing early is still `completed`, the work unit ended), posts `summary` as the closing comment, then sets the task `completed` LAST (custom statuses honoured). You already closed the session by the clock rule → `session: null`, the rest still lands. A timer on another task is left alone.
   - The deliverable still needs a review / merge → `task_status: "in_progress"`, then `tasks_update {id, project_status_id: <review status>}` with the project's review-type status (`projects_statuses_list {id: project_id}`); the project has none → add it once with `projects_statuses_create {project_id, name: "In review", category: "in_progress"}` (owner / team admin only — refused with `forbidden` → say so and leave the task in progress), and say so.
   - A project or team write refused with `inherited_palette` (the board follows a higher level) → SAY SO AND STOP: changing an inherited board is a DECISION for the user, never taken unasked — even when the user manages that level. Tell them the board is inherited (the refusal names the level) and offer the ways forward: customise the project's own statuses (`projects_statuses_customize {project_id}` — the step the refusal names), or, if they manage the level above, change the palette there (`teams_statuses_customize {team_id}` / `teams_statuses_create`, or `workspaces_statuses_create`/`_update` for the workspace default — the workspace and team levels need a paid plan). Run none of them until the user asks; leave the task `in_progress` meanwhile.
4. Read back: the returned `task.status`; `pomodoro_list {status: "active"}` is empty. Never leave a session `active` to run down on its own.

- Older server without `focus_finish` → the primitives, in this order: `pomodoro_update {id, status: "completed", end_time: <now Z>, elapsed_time: <real seconds>, completion_notes: "<done>"}` → `comments_create {task_id, content_text: <summary>}` → `tasks_update {id, status: "completed"}` (or the completed `project_status_id`). Comment FIRST, status LAST.

## Something else is broken

A bug, gap or contradiction you run into that is NOT the task you are working (a tool that does not enforce what its docs promise, a wrong label, a crash on a side path) gets its own task in the workspace's **`Agent findings`** inbox — never a mental note, never a fix smuggled into the current change. The user triages the inbox and moves each task to the project it belongs to (UI, or `tasks_update {id, project_id}`).

1. Dedupe: `tasks_search {q: "<key words of the symptom>"}` — an open task already covers it? Reference its code instead of filing twice.
2. Inbox: `projects_agent_inbox {}` — one call, get-or-create, race-safe; the server picks the reserved key (`AGF`, or `AGF2`… when a legacy project holds it). Pass `workspace_id` (from `projects_get {id: project_id}` of the task you are working) when that task lives in a company workspace; omit for personal. `created: true` → tell the user, in one line, that you created it. Older server without the tool → `projects_list {}` → the project NAMED `Agent findings` (case-insensitive; the name is the identity, never the prefix); missing → `projects_create {name: "Agent findings", prefix: "AGF", description: "Bugs and gaps filed by AI agents while working tasks. Triage here, then move each task to its project."}`; refused with `duplicate_prefix` → retry with `AGF2`, then `AGF3`. Never `projects_create` the inbox on a server that has the tool: `AGF` is a reserved key there (`reserved_prefix`).
3. `tasks_create {title: "<the fact: one sentence, normally under 120 chars, never above 200>", description: "<HTML (or marked text) body written as the restivity-task-writing skill says: Esempio reale (how to reproduce) → Causa → Proposta → Dove vive → Trovato da (you, while working <code>) → Stima>", project_id: <inbox>, priority}` — it comes back as `AGF-n`. Reproduce only where it writes no real data (local / staging), and say so in the description.
4. Priority by blast radius: `urgent` = already happening to real users, security, data loss · `high` = wrong data or behaviour possible on a real path · `medium` = docs / contract drift, degraded UX · `low` = cosmetic.
5. Name it in the `summary` of `focus_finish` ("found AGF-n", with the task URL — the move to its final project re-codes it).
6. Keep working your task. The finding is filed; it is not yours to fix now unless the user says so.

## Interrupted

| The user says                         | Call                                                                                                                                                      | Task                                             |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| "wait"                                | `pomodoro_update {id, status: "paused", elapsed_time}`; resume with `focus_start {task}` (or `status: "active"`)                                          | unchanged                                        |
| "stop"                                | `focus_finish {task, summary: "<hand-off: state, next step>", session_status: "interrupted", task_status: "in_progress"}`                                 | stays `in_progress`, the summary IS the hand-off |
| "drop it"                             | `pomodoro_update {id, status: "abandoned"}`                                                                                                               | ask whether to move it back to `todo`            |
| hard stop, no chance to call anything | next turn: `pomodoro_list {status: ["active", "paused"]}` → close the stale one as `interrupted` at your last known timestamp and say it is reconstructed | —                                                |

## Session states

`active` running · `paused` · `completed` = the work unit ended (counts for `completed_pomodoros`, streaks, badges, earnings) · `interrupted` = stopped before done · `abandoned` = discarded. Never delete sessions. `session_frozen` means the session is already invoiced — leave it alone. A session left `active` past its planned end is auto-closed by the server as `interrupted` — that is a recovery, not a way to finish: close your own blocks.

## Common mistakes

- Starting the session before the task is in progress, or completing the task without the comment — `focus_start` / `focus_finish` exist so the order cannot go wrong.
- Using `focus_start` to log time already spent (that is `pomodoro_create` with `start_time`), or `focus_finish` for a pause (that is `pomodoro_update {status: "paused"}`).
- `elapsed_time` in minutes (it is seconds); `end_time` without the trailing `Z`.
- Calling `profile_get` to pick a duration; passing a task code where a UUID is required (`pomodoro_create`, `comments_create`).
- Logging breaks you did not take; leaving a session `active` after the work stopped.
- Opening a second work session while one is still `active` or `paused` — two timers on one person.
- Reporting a bug you found only in chat: a week later nobody remembers it — file the task.
