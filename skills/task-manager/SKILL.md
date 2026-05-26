---
name: task
description: Manage project tasks stored as Markdown files in .tasks/. Use for all task operations — status check, create new tasks, start/complete tasks, log progress, create and manage subtasks. Invoke when the user runs /task or asks to manage tasks.
argument-hint: '(none) status | new <title> | next | done | wontdo <id> | backlog | log <msg> | sub new/next/done/log'
---

You are the task manager for this project. All tasks live in `.tasks/` at the project root.

**Arguments:** $ARGUMENTS

---

## Step 1 — Initialize if needed

Check whether `.tasks/BACKLOG.md` exists. If it does not, create the full directory structure now before doing anything else:

1. Create the directory `.tasks/tasks/`
2. Create `.tasks/BACKLOG.md` with this exact content:
   ```
   # Backlog

   Ordered by priority (highest at top). One line per task.

   | Task | Summary | Priority | Effort |
   |------|---------|----------|--------|
   ```
3. Create `.tasks/IN_PROGRESS.md` with this exact content:
   ```
   # In Progress

   Only one task at a time.

   | Task | Summary | Priority | Effort | Started |
   |------|---------|----------|--------|---------|
   ```
4. Create `.tasks/DONE.md` with this exact content:
   ```
   # Done

   Most recently completed at top.

   | Task | Summary | Completed |
   |------|---------|-----------|
   ```
5. Add `.tasks` to `.gitignore` 

---

## Step 2 — Route by subcommand

Parse the first word of $ARGUMENTS as the subcommand. If the first word is `sub`, parse the second word as the sub-subcommand. Execute exactly one branch:

---

### (no arguments) — Status

Read `.tasks/IN_PROGRESS.md` and `.tasks/BACKLOG.md`. Report:
- **Active task:** task ID, title, Ref (if present), Started date, and last 3 Progress Log entries. If an `**Active Subtask:**` field is set, open that subtask file and show its title and last 3 Progress Log entries too.
- **Up next:** first 3 rows from BACKLOG (task ID + summary).

If no active task and no backlog items, say so.

---

### `new <title>` — Create a new task

Arguments after `new` form the title, with an optional external reference prefix.

**Detecting an optional Ref:** If the first word after `new` looks like an external ticket ID (matches the pattern `WORD-digits`, e.g. `PROJ-123`, `ENG-42`) or is a URL (starts with `http`), treat it as the `Ref` and the remaining words as the title. Otherwise treat all words as the title and omit the Ref field.

1. Scan all rows in `.tasks/BACKLOG.md`, `.tasks/IN_PROGRESS.md`, and `.tasks/DONE.md` to find the highest TASK-NNN number. The new task gets the next number (padded to 6 digits).
2. Create `.tasks/tasks/TASK-NNN.md` using this template (fill in values; include `**Ref:**` line only if a ref was detected):
   ```
   # TASK-NNN: <title>

   **Status:** Backlog
   **Priority:** Medium
   **Effort:** M
   **Ref:** <ref>          ← omit this line if no ref was provided
   **Created:** YYYY-MM-DD

   ---

   ## Description

   What needs to be done and why.

   ## Acceptance Criteria

   - [ ] ...

   ## Notes

   Any extra context, links, or decisions already made.

   ---

   ## Progress Log

   <!-- Claude appends entries here as work progresses -->
   ```
3. Add a row to the end of the BACKLOG table: `| [TASK-NNN](tasks/TASK-NNN.md) | <title> | Medium | M |`
4. Confirm: "Created TASK-NNN: [title] — added to backlog." Include the Ref if one was stored.

---

### `next` — Start the next backlog item

1. Read `.tasks/IN_PROGRESS.md`. If the table has a row, stop — show the active task and tell the user to finish it first.
2. Read `.tasks/BACKLOG.md`. Take the first row from the table.
3. Open that task file. Set `**Status:** In Progress` and add `**Started:** YYYY-MM-DD` (today's date).
4. Remove that row from the BACKLOG table.
5. Add that row to the IN_PROGRESS table (append a Started column with today's date).
6. Confirm: "Started TASK-NNN: [title]"

---

### `done` — Complete the current task

1. Read `.tasks/IN_PROGRESS.md`. If the table has no rows, stop — nothing is in progress.
2. Open the task file. If it has a `## Subtasks` section, check whether any row in the subtasks table has a Status other than Done. If so, list the incomplete subtasks and warn the user — do not proceed unless they confirm.
3. Note the task ID and summary from the IN_PROGRESS table row.
4. Set `**Status:** Done` and add `**Completed:** YYYY-MM-DD` (today's date) in the task file.
5. Remove the row from IN_PROGRESS.
6. Add a row at the top of the DONE table: `| [TASK-NNN](...) | summary | YYYY-MM-DD |`
7. Confirm: "Completed TASK-NNN: [title]"

---

### `wontdo <TASK-ID>` — Dismiss a backlog task

The task ID is the first word after `wontdo` (e.g. `TASK-003`).

1. Find the row for that task ID in `.tasks/BACKLOG.md`. If not found, stop — report that the task isn't in the backlog.
2. Open the task file. Set `**Status:** Won't Do` and add `**Completed:** YYYY-MM-DD` (today's date).
3. Remove the row from BACKLOG.
4. Add a row at the top of the DONE table: `| [TASK-NNN](...) | summary | YYYY-MM-DD ⊘ Won't Do |`
5. Confirm: "Dismissed TASK-NNN: [title]"

---

### `backlog` — Full backlog overview

Read `.tasks/BACKLOG.md`. For every row in the table, open the corresponding task file and report a summary grouped by Priority (High → Medium → Low), showing:
- Task ID + title, and Ref if present
- Priority and Effort
- First line of the Description section

If the backlog is empty, say so.

---

### `log <message>` — Add a progress entry to the active task

The message is everything in $ARGUMENTS after the word "log".

1. Read `.tasks/IN_PROGRESS.md`. If the table has no rows, stop — nothing is in progress.
2. Open the task file.
3. Append this line to the Progress Log section: `- YYYY-MM-DD: <message>`
4. Confirm: "Logged to TASK-NNN."

---

### `sub new <title>` — Create a subtask under the active task

The title is everything after `sub new`.

1. Read `.tasks/IN_PROGRESS.md`. If no active task, stop.
2. Open the parent task file. Count existing rows in the `## Subtasks` table (if present) to determine the next subtask number M (1-indexed and padded to 3 digits).
3. Create `.tasks/tasks/TASK-NNN-M.md` using this template:
   ```
   # TASK-NNN-M: <title>

   **Status:** Backlog
   **Created:** YYYY-MM-DD

   ---

   ## Description

   What needs to be done and why.

   ## Acceptance Criteria

   - [ ] ...

   ## Notes

   ---

   ## Progress Log

   <!-- Claude appends entries here as work progresses -->
   ```
4. If the parent file does not yet have a `## Subtasks` section, append it after the Notes section:
   ```
   ## Subtasks

   | Subtask | Summary | Status |
   |---------|---------|--------|
   ```
5. Add a row to the Subtasks table: `| [TASK-NNN-M](TASK-NNN-M.md) | <title> | Backlog |`
6. Confirm: "Created subtask TASK-NNN-M: [title]"

---

### `sub next` — Start the next unstarted subtask

1. Read `.tasks/IN_PROGRESS.md`. If no active task, stop.
2. Open the parent task file. Find the first row in the `## Subtasks` table with Status `Backlog`.
   If none found, report all subtasks are done or in progress.
3. Open that subtask file. Set `**Status:** In Progress` and add `**Started:** YYYY-MM-DD`.
4. Update its row in the parent's Subtasks table: change Status to `In Progress`.
5. Set or replace the `**Active Subtask:**` field in the parent task file (add it after the last metadata field if not present).
6. Confirm: "Started subtask TASK-NNN-M: [title]"

---

### `sub done` — Complete the active subtask

1. Read `.tasks/IN_PROGRESS.md`. If no active task, stop.
2. Open the parent task file. Read the `**Active Subtask:**` field. If not set, stop — no subtask is active.
3. Open the subtask file. Set `**Status:** Done` and add `**Completed:** YYYY-MM-DD`.
4. Update its row in the parent's Subtasks table: change Status to `Done`.
5. Remove the `**Active Subtask:**` line from the parent task file.
6. Confirm: "Completed subtask TASK-NNN-M: [title]". If all subtasks are now Done, note that the parent task is ready to close.

---

### `sub log <message>` — Log progress on the active subtask

The message is everything after `sub log`.

1. Read `.tasks/IN_PROGRESS.md`. If no active task, stop.
2. Open the parent task file. Read the `**Active Subtask:**` field. If not set, stop — no subtask is active.
3. Open the subtask file. Append to its Progress Log: `- YYYY-MM-DD: <message>`
4. Confirm: "Logged to TASK-NNN-M."
