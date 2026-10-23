---
name: restivity-task-writing
description: "Use when an agent CREATES or REWRITES a Restivity task (tasks_create / tasks_update description) — a bug, a finding, a feature request, a chore — so the card reads like the house ones: a title that states the fact, then Esempio reale → Causa → Proposta → (Consiglio) → Dove vive → Trovato da → Stima, each section a paragraph that opens with its bold label and a period (no headings), as TipTap HTML (or marked text the server renders), in the user's language."
metadata:
  version: '1.2.2'
---

# Restivity task writing

A task on a Restivity board is read by someone who was not in your conversation: the owner triaging the board, a teammate picking it up next week, another agent. The card must let that person act without asking you anything: what happens, to whom, where it lives in the code or the product, what to do about it, who found it, how long it takes.

This skill covers the **body of a new task** (and a full rewrite of one). Comments, closing summaries and progress notes ON a task follow `restivity-task-updates`.

> **This binds every client that creates a task through MCP or REST** — Claude Code, claude.ai's
> connector, Cursor, a script. If you can read this file, it applies to you.

**The user's request wins.** This shape is the default for when the user has not said otherwise: if they ask for another structure, add or drop a section, want a shorter card, another language or no estimate, do what they ask and never argue the template back to them. On the house board the rule reads: «Salvo richieste diverse dell'utente, la scheda segue questa forma.»

## The title states the fact

- **The fact, not a category.** `Creazione team: se la riga del proprietario fallisce, resta un team senza proprietario` — not `Bug creazione team`, not `Problema con i team`.
- **No prefix** — no `Bug:`, `Fix:`, `[Security]`, `TODO`. The project, the priority and the tags already classify the card. The one prefix the house uses is `Decisione da confermare:` (a choice an agent made without the owner, which the owner must confirm or overturn).
- **One sentence: who / what / under which condition.** Normally under 120 characters and never above 200 — the condition is what makes the fact true, so it stays; cut adjectives before you cut the condition.
- **No code of its own.** The task code (`AGF-184`, `RES-58`) is assigned by the system from the project's prefix — never write one yourself, and never put the code of the task you are creating in its own title. Mentioning _another_ task's code (`follow-up di AGF-329`) is fine.

## The six sections

Each section is one paragraph that opens with its label in bold followed by a period — `<p><strong>Causa.</strong> …</p>` in HTML, `**Causa.** …` in marked text — and not with a heading (`###`, `<h3>`); the labels run Esempio reale → Causa → Proposta → (Consiglio) → Dove vive → Trovato da → Stima, and Consiglio, which option the author recommends, appears only on a card that sets out options for the reader to choose. The label is part of the paragraph, so the board reads short paragraphs, each starting in bold: on AGF-510 (22/10) the labels went in as `###` headings and the board showed them as detached titles, and the owner asked for them in bold. Use the labels in the user's language:

| Italian (house board)       | English                          |
| --------------------------- | -------------------------------- |
| **Esempio reale.**          | **Real example.**                |
| **Causa.**                  | **Cause.**                       |
| **Proposta.**               | **Proposal.**                    |
| **Consiglio.** _(optional)_ | **Recommendation.** _(optional)_ |
| **Dove vive.**              | **Where it lives.**              |
| **Trovato da.**             | **Found by.**                    |
| **Stima.**                  | **Estimate.**                    |

1. **Esempio reale** — one concrete scene the reader can replay: a named person with their role (`Giulia, membro del workspace`; the house uses first names such as Sara, Giulia, Luca, Marco — never a real customer's name or email), the screen, the click, **what happens today** and **what should happen**. When it was measured on a real environment, say where and give the numbers (`13/10, produzione: 10 connessioni su 11`). Not: "users may experience issues", "in some cases".
2. **Causa** _(bugs and findings)_ — **where** it lives (file and function, policy, trigger, migration number, route) and **why it slips through**: why the code believes it succeeded, or why the tests stayed green (`la DELETE non trova righe e non dà errore`). If you have not found the cause, say `Causa: non ancora trovata` and what you ruled out — never guess one.
3. **Proposta** _(or **Fix** / **Correzione** once it is fixed)_ — what to do, concretely enough to start. For code: the **RED-first test** that proves it — the scene, what it shows today (the proof), what it shows after, and the read-back that confirms the write (`oggi il team resta (prova), dopo non c'è più alla rilettura admin`). When there is a real alternative, one line for it.
   - **Consiglio** _(optional — only on a card that sets out options for the reader to choose)_ — its own paragraph right after the Proposta: which option the author recommends and why, in one or two sentences (`<p><strong>Consiglio.</strong> A: mezza giornata, e chiude l'ultima voce «SHOULD» della RFC ancora aperta.</p>`). Not inside the Proposta paragraph, and not on a card with one proposal.
4. **Dove vive** — the places to open, in `code`: paths, functions, policies, migrations, decision ids (`D-F5.4`), PR numbers. A list when there are more than three.
5. **Trovato da** — who found it and the state. A person by **role**, never by email (`l'owner durante il test manuale`, `un utente del piano Team`); an agent as `Claude (Opus, review Claude)` or the client and model that really found it. Then the state: `Non corretto.` / `Corretto nella PR #372.` / `Da chiudere dopo la release in produzione.` Add the date (`il 13/10`) when the finding is time-sensitive.
6. **Stima** — in concrete units: `2 ore`, `mezza giornata, una migrazione`, `2 giorni`. Never "small", "quick", "a bit". When the work is already done, say what is left (`nessun lavoro rimasto: si chiude dopo la release`).

**What a section must NOT contain:** narration of your own steps ("I first looked at…"), praise or apologies, hedging, restating the title, a paragraph longer than three sentences. Numbers, paths and codes go where the reader scans — in the paragraph's first line or in a list — not buried in prose.

### Which sections drop for which kind

| Kind                              | Sections                                                                                                                                                                           |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Bug / finding**                 | All six; **Consiglio** after the Proposta when it sets out options for the reader to choose.                                                                                       |
| **Feature request**               | Esempio reale (the scene the user wants to live) → Proposta → (Consiglio) → Dove vive (if known) → Trovato da (who asked) → Stima. **No Causa** — nothing is broken.               |
| **Chore** (upgrade, cleanup, ops) | Proposta → (Consiglio) → Dove vive → Trovato da → Stima. Esempio reale only when it helps (what breaks if it is not done). No Causa.                                               |
| **Decisione da confermare**       | Esempio reale (the choice as the user meets it) → the choice made and why → **L'alternativa** → (Consiglio) → Dove vive → who made it. No Stima unless the alternative has a cost. |

Never pad a section to make the shape complete: a missing section is better than an invented one. Never invent a fact (a file, a number, a person's role) to fill one either — say what is unknown.

## Project, priority, tags

- **Project.** The project the user names. A finding you meet while working goes to the workspace's **agent inbox** (`projects_agent_inbox` gives its id; the first time there is none, it is the person who chooses one — the `restivity-focus-flow` skill) unless the user says otherwise. `project_id` is a UUID: resolve a name with `projects_list`.
- **Priority** (`low` · `medium` · `high` · `urgent`) — the user's word wins. Otherwise: `urgent` only for something breaking for users right now (data lost, people locked out); `high` when a person gets a "yes" they should not get, money is wrong, or a main flow is broken; `medium` for a bug with a workaround (the default); `low` for cosmetic issues, chores and decisions to confirm. Leave `is_urgent` / `is_important` unset unless the user asks — they place the task on the Eisenhower matrix, they are not a second priority.
- **Tags.** Only tags that already exist: read them with `tags_list` and pass their ids in `tag_ids`. **Never create a tag** to classify a task unless the user asked for that tag.
- **Assignee.** Leave `assigned_to` out and the task is assigned to the person you act for — the task is theirs, which is what a finding you file in their name should be. Name a teammate's id only when the user names them (a member of the task's project; anyone else is refused), and send `assigned_to: null` only when the user wants the task unassigned.
- **Names that look alike — ask, never guess**: when the user names a client, project, team or tag that resembles two or more existing ones (the same words in another case or spacing, one a typo away from another, one name inside another — «Sito» and «Sito web»), or the name is otherwise ambiguous, ask which one they mean before acting, list the candidates, and act only on their answer — never pick one yourself, and never create a new one because no match was exact. Names are unique where a duplicate would confuse (a team's projects, a workspace's teams and team-less projects, a person's tags, a workspace's clients — compared without case and surrounding spaces): a create or rename onto a taken name is refused with `name_taken`; tell the user and offer the existing one instead of retrying with a variant of the name.

## The description: HTML, or marked text — the server renders it; never raw Markdown on the board

`description` on `tasks_create`, `tasks_update`, `tasks_batch_update`, `tasks_recurrence_update_rule` and `POST /api/v1/tasks` is a **string**, at most 10,000 characters. It is **not** a TipTap JSON document (an object is refused with `validation_error`). The contract, word for word as every tool states it:

> `description` takes HTML or marked text, at most 10,000 characters. HTML is kept as the task editor writes it (`<p>`, `<strong>`, `<em>`, `<ul>`/`<ol>` with `<li>`, `<code>`, `<a href="https://…">`) and sanitized by allow-list; marked text — blank-line paragraphs, `- ` and `1. ` lists, a `### ` heading opening a block, `**bold**`, `*italic*`, `` `code` ``, `[text](https://…)` links, code blocks fenced by three backticks — is rendered by the server into that HTML, so the board shows no asterisks or backticks. In one text the two do not mix: a text holding an HTML tag is stored as sent, markers included. A read returns the HTML in `description` and the same content without tags in `description_text`.

So you have two ways to write it, and the person reading the board sees the same card either way — bold labels, real lists, never an asterisk or a backtick:

- **HTML**, the format the task editor saves — the worked example below is HTML. Prefer it when you compose a full card.
- **Marked text**, when HTML is awkward for you: blank lines between sections, `**Causa.**` for a label, `- ` or `1. ` in front of list items, `` `code` `` for a path. The server renders it into the same HTML before storing it (and does the same for a comment's `content_text`: one grammar for both fields).

**Never mix the two** in one text: a text that already holds an HTML tag is stored as sent, so `<p>**Causa.**</p>` keeps its asterisks on the board. A conversion that would exceed 10,000 characters is skipped and the text is stored as sent. When you read a task back to show it to a person, show `description_text` (the same content without tags), never the HTML.

The server sanitizes the HTML against an allow-list before storing it. Use only:

- `<p>` for each section, `<strong>` for its label;
- `<ul>` / `<ol>` with `<li>` for lists;
- `<code>` for paths, functions, policies, codes; `<a href="https://…">` for a PR or a page;
- escape `<`, `>` and `&` in your text as `&lt;`, `&gt;`, `&amp;`.

Anything else (scripts, event attributes, unknown tags, `data:` URLs) is stripped.

### Worked example

A real house card (AGF-184, a bug fixed in the FIX-PLAN family 5, decision D-F5.4), sent to `tasks_create`:

```json
{
  "title": "Creazione team: se la riga del proprietario fallisce, resta un team senza proprietario e nessuno lo gestisce",
  "project_id": "<the agent inbox's id, from projects_agent_inbox>",
  "priority": "medium",
  "description": "<p><strong>Esempio reale.</strong> Giulia crea il team «Marketing». Il team viene salvato, ma la riga che la rende proprietaria fallisce (un limite di posti, un vincolo, una connessione caduta). L'app prova a cancellare il team appena creato e non ci riesce: resta un team «Marketing» senza proprietario, che nessuno può gestire né rimuovere dall'app. Atteso: il team nasce con la sua proprietaria, oppure non nasce.</p><p><strong>Causa.</strong> L'annullamento in <code>createTeam</code> cancella il team con il client di Giulia. La policy <code>teams_delete</code> ammette solo il proprietario del team, e la riga che fa di Giulia la proprietaria è proprio quella fallita: la DELETE non trova righe e non dà errore, quindi il codice crede di aver annullato. Il test della migrazione 196 documentava già questa forma.</p><p><strong>Proposta.</strong> L'annullamento gira con il ruolo di servizio, limitato a quel team e a chi l'ha creato (<code>created_by</code>); un errore dell'annullamento viene loggato. Test RED-first: un trigger fa fallire apposta la riga del proprietario per un nome di team; oggi il team resta (prova), dopo non c'è più alla rilettura admin. Controllo: lo stesso creatore, senza trigger, crea il team.</p><p><strong>Dove vive.</strong> <code>src/lib/services/teamService.ts</code> (<code>createTeam</code>, l'annullamento) · policy <code>teams_delete</code> · D-F5.4.</p><p><strong>Trovato da.</strong> Claude (famiglia 5 del FIX-PLAN). Corretto nella PR #372, sentinella S382.</p><p><strong>Stima.</strong> Nessun lavoro rimasto: si chiude dopo la release in produzione.</p>"
}
```

The same body, for a reader, renders as six short paragraphs with bold labels — the shape of every card on the house board.

## Before you send

- [ ] The title states the fact in one sentence (who / what / under which condition — normally under 120 characters, never above 200), with no prefix and no task code of its own.
- [ ] **Esempio reale** names a person by first name or role, the screen, the click, what happens today and what should happen.
- [ ] **Causa** names the place (file, function, policy, migration) and why it slips through — or says it is not yet found.
- [ ] **Proposta** says what to do and, for code, the RED-first test with its read-back.
- [ ] **Trovato da** names who found it (a role, or the agent and model) and the state: fixed or not.
- [ ] **Stima** is in hours or days.
- [ ] Every section is a paragraph whose first words are its bold label and a period (`<strong>Causa.</strong>`, `**Causa.**`) — no `###`, no heading; **Consiglio** only where the Proposta sets out options.
- [ ] `description` is HTML (`<p><strong>…</strong> …</p>`) or marked text — never JSON, never the two mixed in one text.
- [ ] Tags exist already; an ambiguous name was asked about, not guessed.
- [ ] Everything is in the user's language.
