# Restivity for Claude

[Restivity](https://restivity.ai) keeps your tasks, a focus timer and your earnings in one calm place.
This plugin connects Claude to your Restivity account and teaches it to work there the way the app expects:
tasks written like the ones you write, progress notes that stay short, focus sessions that are tracked.

## What is inside

- **The Restivity connector**: one remote MCP server, `https://restivity.ai/api/mcp` (streamable HTTP).
  Its tools read and change your tasks, projects, custom statuses, teams, focus sessions, comments, tags and profile, and read your earnings.
- **4 skills**, Markdown instructions Claude reads before it acts:
  - **`restivity-automation`**: Drive Restivity (tasks, projects, pomodoro, tags, comments) as an AI agent via its MCP server and REST API — auth, the tool catalog, and the key task/today/search/batch/recurrence/subtask workflows.
  - **`restivity-focus-flow`**: Use when an agent is asked to actually work on a Restivity task (implement, fix, write, research it) rather than only read or edit it, and the effort must show up in Restivity as task status, pomodoro work sessions and a closing comment. Triggers: 'work RES-58', 'lavora il task', 'take this task', 'track my time on it', an overlapping_active_session error.
  - **`restivity-task-updates`**: Use when an agent writes ON a Restivity task through MCP or REST — a comment, a closing summary, a task description or an Agent-findings report — so the update is short, structured (TipTap paragraphs and lists, never a wall of text) and says only what the reader needs: what changed, why, what is next.
  - **`restivity-task-writing`**: Use when an agent CREATES or REWRITES a Restivity task (tasks_create / tasks_update description) — a bug, a finding, a feature request, a chore — so the card reads like the house ones: a title that states the fact, then Esempio reale → Causa → Proposta → (Consiglio) → Dove vive → Trovato da → Stima, each section a paragraph that opens with its bold label and a period (no headings), as TipTap HTML (or marked text the server renders), in the user's language.

## Requirements

The connection needs a Restivity Pro or Team plan; a paid Team seat also unlocks it on a free personal plan. A free account can sign in, and the tools then answer that its plan does not include agent access.

## Install and sign in

- **Claude Code** 2.1.275 or later, in one step:

  ```text
  /plugin install restivity --marketplace Restivity/restivity-claude-plugin
  ```

  On an earlier version, add the marketplace first, then install from it:

  ```text
  /plugin marketplace add Restivity/restivity-claude-plugin
  /plugin install restivity@restivity
  ```

  Then run `/mcp`, pick `plugin:restivity:restivity` and choose **Authenticate**.
  To try a local clone instead: `claude --plugin-dir ./restivity-claude-plugin`.
- **Claude** on the web and in the desktop app: Customize → Plugins → Add → Add marketplace, paste https://github.com/Restivity/restivity-claude-plugin and add Restivity. In Cowork: + → Plugins → Manage plugins → Add marketplace → Add from a repository, paste the same address and install Restivity. Then open its **Connectors** tab and connect Restivity.

Either way you sign in to Restivity in your browser and approve access: there is no API key to paste.
You can revoke the access at any time from Settings → Connected apps in Restivity (https://restivity.ai/settings/integrations).
The full walkthrough, for Claude and for other agents, is at https://restivity.ai/learn/agents.

## What it runs, sends and fetches

- **Runs**: nothing on your computer. The plugin has no hooks, commands, scripts or binaries; the skills are text.
- **Sends**: the requests Claude makes with the Restivity tools, and only when it uses them,
  to `https://restivity.ai/api/mcp` over HTTPS, signed in with OAuth 2.1.
  What Restivity does with your data is in the privacy policy: https://restivity.ai/privacy.
- **Fetches**: nothing else. The skills ship inside the plugin.
  The current text of each one is also served at https://restivity.ai/.well-known/agent-skills/index.json, with its version and digest.

## Support

Write to support@restivity.ai. Terms of service: https://restivity.ai/terms.

## License

The skills and the configuration files are released under the MIT License (see `LICENSE`).
The license grants no right to the Restivity name or logo.

This repository is generated from the Restivity application repository: changes are made there, not here.
