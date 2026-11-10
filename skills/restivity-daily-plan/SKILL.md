---
name: restivity-daily-plan
description: "Use when a person asks the agent to run their day on Restivity (plan my day, what do we close today, add this to today, recap): one task per calendar day carries what the day closes as its subtasks, a timed log of what happened when in its description, and the recap in its comments, in the person's language and timezone."
metadata:
  version: '1.0.0'
---

# Restivity daily plan

One card on the person's board holds the day: what today closes, as subtasks; what happened and when, as a timed log in the description; the recap, as comments. The person says «plan my day», «what do we close today», «add this to today», «recap», and the agent keeps that one card true from the first request of the day to its close. The card is not a diary and not a catch-all: it says what the day CLOSES, and everything else stays off it.

The person chooses what enters their day. The agent never grows the day alone: a day that grows with every request never closes, and Restivity exists to let a person work calmly while keeping the choice theirs.

## The day card

1. A day is a calendar day in the PERSON's timezone. Read it once with `profile_get` (it answers the `timezone`), then take today's date and the time of day from a real clock (a shell's `date -u`, or the current time your client gives you) converted to that zone. Never guess either: a log line with a wrong time is worse than no line.
2. Before creating, look for today's card: `tasks_search {q: "<the date as the title spells it>"}` matches task titles and descriptions, with typo tolerance; `tasks_today` answers the focus list for today (the tasks flagged `is_for_today`, and the overdue ones), in the person's timezone. Either can find the card. Typo tolerance means an older day card can match too: read the hit's date (the title, the `due_date`) before calling it today's. **Never two cards for one day**: one exists, work it.
3. No card yet: ask the person ONCE where their day cards live (one project, or none: a task without a project is fine), and keep that answer for every day after. Follow the column rule of `tasks_create`: when the person did not say which status, ask which column before creating, the columns come from `projects_statuses_list`; a create sent without asking still succeeds and the answer names the column it went to, so tell the person which column the card is in.
4. Create with `tasks_create`: a title that states the fact, in the person's language, the date in it (`Day 7 Nov: what we close today` / `Giornata 7/11: cosa chiudiamo oggi`), `due_date` today, `is_for_today: true`. The two are independent: set both. Keep the returned task `id` (the UUID): comments want it.

## Subtasks: what the day closes

Each subtask is a thing the day should close, written so the person understands it days later without the conversation that produced it: «Reply to Marta about the invoice», never «that thing from this morning». Three to five subtasks is a day; a card with fifteen open subtasks is not a plan, it is a backlog, and the person should be told so when it happens.

`subtasks` on `tasks_update` REPLACES the whole list. Read the card with `tasks_get` first (it answers the subtasks with their `id`s), send every existing item back with its `id`, and add the new one without an `id`:

```json
{
  "id": "<card>",
  "subtasks": [
    { "id": "<existing-1>", "title": "<existing-1 title>" },
    { "id": "<existing-2>", "title": "<existing-2 title>" },
    { "title": "Reply to Marta about the invoice" }
  ]
}
```

The id-less item is created, and it lands at the TOP of the list. The order of the list is the order of the day: the top subtask is what closes first, and an urgent addition goes above it only because the person said so. A list sent without the `id`s deletes every subtask the card had and recreates it new: which were done is lost. A subtask renamed in place keeps its `id` and its done state.

A subtask is checked only when the work is finished AND verified: the reply was sent, the test ran, the artefact was read back. Never from memory, and never from a report you did not check yourself. If the work is half done, the subtask stays open and the log says where it stopped.

## The description is the log

The card's `description` holds what happened, when:

> How to read this: one line per event, the time it happened and one sentence.
>
> 09:12 Opened the day: 3 things to close.
> 10:40 Invoice reply sent (subtask checked).
> 14:05 Marta asked for the contract too: placed, she chose tomorrow morning.

`description` is one field, so sending it replaces the whole text: read the card (`tasks_get`), append the line, send it all back (`tasks_update`). Write it as the `restivity-task-writing` skill says a description is written (marked text or HTML, bold labels, never the two mixed), and mind the limit it states: 10,000 characters. When the log nears it, fold the oldest lines into one line («morning: 4 events, the invoice and the reply closed») and say so in the log itself.

## A new request during the day

A request that arrives mid-day is PLACED before it is done, and the person chooses whether it enters today. No question when there is no choice to make: a question about the work, a correction, something finished inside the reply. Those are answered, not scheduled.

1. Read the day's load first: the open subtasks of the card, the time left in the person's day, what is running (a work session on another task, a blocked item).
2. Propose one of two, the recommendation first:
   - «today», when it is urgent, unblocks something already open, is the same family as the open work, or is very short;
   - «later», naming when, and what doing it now would push out of today.
3. Ask ONE short question («Today it is 4 pm and two things are still open: this looks like tomorrow morning, it would push out the invoice. Today or tomorrow?» / «Sono le 17 e sono aperti due punti: lo metto a domani mattina, sposterebbe la fattura. Oggi o domani?»). The person's answer wins, always, without arguing twice; «urgent» means today, at the top of the subtask list. Their answer may be neither proposal («next week»): that is a later, with the time they named.
4. Record the choice. Today: a subtask (the whole-list rule above) plus a log line. Later: a log line, and a subtask on the future day's card, or its own task with a `due_date` (`tasks_create`), named so the person finds it.

## The recap is the comments

Whenever something closes, or there is news the person must know, post ONE comment with `comments_create` (it takes the card's task `id`, the UUID, not the code), shaped as the `restivity-task-updates` skill says: a bold lead with the outcome, one short list, a closing line. That skill is the contract for every comment; this one does not restate it. When the person asks for a recap, read the comments with `comments_list {task_id, sort_by: "created_at", sort_order: "asc"}` and answer from them: what closed, what moved, what waits for the person. Never answer a recap from memory: the comments are the day's memory.

A bug or gap met while working the day is not a subtask of the day: it goes to the workspace's agent inbox as its own task, the way `restivity-focus-flow` describes, and one log line says it was filed.

## The end of the day

- One last log line: what closed, what moves to tomorrow, and why.
- Every subtask not checked is carried to the next day's card (created if it does not exist yet, by the rules above), its title marked `(from yesterday)`; nothing silently disappears, and nothing is checked just to close the card.
- The card is completed (`tasks_update {status: "completed"}`) only when every subtask is checked or carried.
- One last comment with the day's recap, as above. Tomorrow's card starts with what was carried, and the person reads the day that ended where the next one begins.
- Tomorrow's card is born at the end of this day (or when the person asks for tomorrow), never at midday: the day being worked is the only card that exists.

## Never

- Secrets or credentials on the card: keys, tokens, passwords, personal data the person did not put there.
- A subtask checked without verifying the work.
- A second card for the same day.
- The person's day written in a language other than theirs.

## Common mistakes

- Adding the new subtask without the existing `id`s: the whole list is rebuilt, the done state is lost.
- Logging an event at the time it is written instead of the time it happened, or guessing the person's clock.
- Answering «recap» from memory instead of `comments_list`.
- Asking the today-or-later question with no recommendation, or arguing after the answer.
- Carrying nothing and completing the card anyway: the day's loose ends stay visible until they are carried.
