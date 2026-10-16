---
name: sync
description: Syncs one or all PROJECT_<name>.md queues with the live ticket tool (GitHub, Jira, Trello, …) — deletes rows of closed tickets, updates changed statuses, and asks about anything strange (reopened, moved out of scope, new tickets, unknown statuses). Use when the user types /project-manager:sync or asks to refresh or check a project's status.
argument-hint: "[project file name or path; none = every project file]"
---

# project-manager:sync

Read `../../references/project-file.md` and `../../references/tools.md`. Both paths are relative to this skill's base
directory.

The tool is the truth for **status and state**. The file is the truth for **order**. Sync never reorders existing rows
and never writes to the tool.

## 1. Which files

- With an argument: resolve that one file, following the contract.
- Without one: every project file (at the root, in `.projects/`, and a legacy `PROJECT.md`), each processed on its
  own.

## 2. Fetch

For each file, list the whole scope from `## Source` in one paginated listing. Then look up each file ticket that is
missing from the listing, directly by its key.

## 3. Apply

These changes are applied without asking:

| Case | Action |
|---|---|
| The ticket is closed in the tool | Delete the row |
| Its status differs and is a forward move, or an equal-rank one (for example Backlog → Ready → In progress) | Update the status |
| Its title changed | Update the title |
| The `Ref` ID is missing or changed | Update it |

These cases are collected, then asked about in **one** `AskUserQuestion` call per file. List each case in the
question text, and offer "Apply the suggestions (Recommended)" / "Decide one by one" / "Leave the file as is":

| Case | Suggestion |
|---|---|
| The status moved **backward** (for example In review → Backlog) | Take the tool's status |
| A status name that `## Source` doesn't list (the workflow changed) | Take it, and add it to the status list |
| An open ticket is in the scope but not in the file (new or reopened) | Insert it where `## Order logic` puts it; give the position |
| A file ticket was removed from the scope but is still open | Delete the row |
| A file ticket can't be found (deleted, or no access) | Delete the row, and tell the user which ticket |
| A ticket is in the tool's done column but still open (or the reverse) | Keep the row and flag it, because the two disagree |

Then renumber the queue and save each file.

## 4. Report

For each file, give one line: `<file>: −N closed, ~N status, +N added, N asked`. Add the new next ticket, and stop
there. If a file's queue is now empty, suggest `/project-manager:cleanup <file>`.
