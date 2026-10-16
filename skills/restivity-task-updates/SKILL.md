---
name: restivity-task-updates
description: 'Use when an agent writes ON a Restivity task through MCP or REST — a comment, a closing summary, a task description or an Agent-findings report — so the update is short, structured (TipTap paragraphs and lists, never a wall of text) and says only what the reader needs: what changed, why, what is next.'
metadata:
  version: '1.3.0'
---

# Restivity task updates

An update on a task is read by a person on a phone between two other things. It has to be scannable in five seconds and say only what that person needs to act. Two tools carry updates: `comments_create` / `comments_update` (the thread under a task) and `tasks_create` / `tasks_update` (the description). Both accept rich content; write it as rich content.

> **This binds EVERY client that writes on a task through MCP or REST** — Claude Code, ZCode,
> Cursor, a script, whatever comes next. The format is part of the contract, not a house style:
> on 2026-10-01 an agent posted four perfectly-informative updates as `content_text` walls of
> prose and the owner could not read them. If you can read this file, it applies to you.

## The three shapes

**A closing / outcome comment** (the summary `focus_finish` posts, a finding you close):

1. **One bold lead** — the outcome or the state, in one line (`Chiuso — fix verificato, test 11/11`, `Fatto — PR #314 verso staging`, `Bloccato: manca la chiave API`).
2. **One short paragraph** of context, only if the lead is not enough.
3. **One list** (3–6 items) for the details a reader may want: what changed, numbers, decisions, findings filed. A label in bold helps scanning: `**Fatto:**`, `**Verificato:**`, `**Scartato:**`.
4. **One closing line** starting with a bold label: `Prossimo:` / `Next:`, `Aperto:` / `Open:`, `Decidere:` / `Decide:`, `Per te:`.

**A progress comment** (mid-work, only when the reader must know — see below):

1. An `heading` (level 3) with the state and the date: `Stato 30/09 ~12:00 — 70 task su 126 (PR1).`
2. **One list** (3–6 items) with labeled bullets: `**Fatto:** …`, `**In corso:** …`, `**Per te:** …` (only the labels that have content).
3. Nothing else — no closing line needed, the heading already dates it.

**A production-proof comment** (once per card, after the release that shipped it is live and verified — the owner's rule of 2026-10-15: «un commento su ogni task quando è provato su produzione»):

1. **One bold lead**, always the same words: `Rilasciato in produzione — completato e revisionato, in attesa della tua chiusura` (EN: `Released to production — done and reviewed, waiting for your closure`).
2. **One list** (2–4 items): what the release verification proved (the deploy, the migrations applied and compared with the files, the smoke that passed) and the walk the reader can take (`MANUAL-TESTS §…`).
3. **One closing line**: `Per te:` / `For you:` — the walk the owner can still take.

It goes on every card the release shipped (a fixed bug, a feature, infra), never on a «Decisione da confermare» (the owner closes those by answering) and never before the release itself has been verified. The agent that posts it then CLOSES the card (`status: completed`; `tasks_batch_update` for many, 50 ids per call) — the owner's rule of 2026-10-15 («puoi chiuderlo tu direttamente»): «In Review» is where finished work waits for the owner's review BEFORE production, and a card that is live and carries this comment is done. A «Decisione da confermare» card lives in its own status of that name and stays open until the owner answers.

Say the necessary, nothing more: no narration of your own steps, no restating the task, no praise, no hedging. Numbers go in the list, not in prose. File paths and commit hashes are fine inline in code. Write in the language the task is written in.

## Send it as a document, not as a string

`content_text` is plain text and cannot carry structure — a paragraph glued to a paragraph reads as a wall. **This is the #1 way the rule is broken**: the sender composes four sections, then ships them as one string and the reader gets an unreadable block (measured 2026-10-01 — four updates, all walls). Send the `content` field as a TipTap document (the server derives the plain-text mirror for search and previews):

```json
{
  "task_id": "<uuid>",
  "content": {
    "type": "doc",
    "content": [
      {
        "type": "paragraph",
        "content": [
          { "type": "text", "marks": [{ "type": "bold" }], "text": "Fatto — PR #314 verso staging" }
        ]
      },
      {
        "type": "bulletList",
        "content": [
          {
            "type": "listItem",
            "content": [
              {
                "type": "paragraph",
                "content": [
                  { "type": "text", "text": "Tab Tasks con le viste di /tasks, progetto pinnato." }
                ]
              }
            ]
          },
          {
            "type": "listItem",
            "content": [
              {
                "type": "paragraph",
                "content": [{ "type": "text", "text": "Vista ricordata per progetto (ui_prefs)." }]
              }
            ]
          },
          {
            "type": "listItem",
            "content": [
              {
                "type": "paragraph",
                "content": [{ "type": "text", "text": "e2e: Kanban mostra le colonne custom." }]
              }
            ]
          }
        ]
      },
      {
        "type": "paragraph",
        "content": [
          { "type": "text", "marks": [{ "type": "bold" }], "text": "Prossimo" },
          { "type": "text", "text": ": in prod col prossimo treno." }
        ]
      }
    ]
  }
}
```

Nodes you may use: `paragraph`, `bulletList` / `orderedList` with `listItem`, `heading` (level 3, rarely), `hardBreak`; marks `bold`, `code`. Mention a person with a `mention` node (`attrs: { id, label }`) only when they must act. A task **description** is not a document: `description` on `tasks_create` / `tasks_update` is an HTML string (`<p>`, `<strong>`, `<ul><li>`, `<code>`) or marked text the server renders into that HTML — the same grammar as `content_text` below; never the two mixed (the `restivity-task-writing` skill has the contract). The body of a NEW task (or a full rewrite) follows the `restivity-task-writing` skill — title = the fact, then Esempio reale → Causa → Proposta → Dove vive → Trovato da → Stima; a short edit to an existing description keeps this skill's brevity.

Servers or clients without a `content` field accept `content_text` — and since AGF-76 (2026-10-01) the server PARSES that structure into real nodes (`textToContentDoc`): blank-line-separated paragraphs, a run of `- ` lines into a bulletList, a run of `1. ` lines into an orderedList, `### ` into a level-3 heading, `**bold**`, `*italic*` and `code` inline marks, `[text](https://…)` into a link, a block fenced by three backticks into a code block, a lone newline into a hardBreak (since AGF-377 the same grammar renders a task description). So the plain-text fallback is not second-class: keep the same four-part order, one item per line with `- ` in front of list lines, `**…**` around labels — it renders exactly like the TipTap above. Round-trips through `extractPlainText` line for line.

## When to write, and when not to

- Write **once per outcome**: the plan when you start (`focus_start` posts it), the summary when you finish (`focus_finish` posts it), and one comment when something the reader must know changes in between (blocked, decision needed, scope changed).
- After a production release, the production-proof comment above on each card it shipped — one per card, no sooner.
- Do **not** write progress noise ("tests running", "starting step 3"): the work session carries the time, the comment carries the outcome.
- A bug or gap you meet on the way goes to the **Agent findings** inbox as its own task (`projects_agent_inbox`, then `tasks_create`) written as the `restivity-task-writing` skill says (title = the fact; Esempio reale → Causa → Proposta → Dove vive → Trovato da → Stima) — and one line in your closing comment says it was filed.

## Checklist before sending

- Lead line is bold and states the outcome.
- No paragraph longer than two sentences; no list longer than six items.
- Every number is in the list, every path or hash in code.
- The closing line tells the reader what happens next, or what they must decide.
- It is a `content` document, not a `content_text` string.
