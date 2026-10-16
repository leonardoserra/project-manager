---
name: next
description: Read-only. Tells briefly what the next ticket in a PROJECT_<name>.md queue is about — title, link, status, problem, done-when, dependencies — without starting work. Use when the user types /project-manager:next or asks "what's next" on a project file.
argument-hint: "[project file name or path]"
---

# project-manager:next

Read `../../references/project-file.md` and `../../references/tools.md`. Both paths are relative to this skill's base
directory.

1. **Resolve the file** from `$ARGUMENTS`, following the contract. When several files exist and no argument is given,
   ask which project.
2. **Pick the next row** following the contract (§ Picking "next").
3. **Check it is still open**: look up that one ticket in the tool named in `## Source`.
   - If it was closed, delete its row, renumber the queue, say so, and move to the next row.
   - If its status changed, update the row.
   - These are the only writes this skill makes.
4. **Read the ticket**: its body and labels, and its comments only if the body is thin.
5. **Answer** in about 8 lines, with no headers:
   - `<Ticket> <title>`: the link, the status, and the labels or priority;
   - what is wrong or wanted, in one or two sentences;
   - "Done when": the acceptance criteria, condensed;
   - its dependencies (still open or already closed) and any open decision the ticket says needs the user;
   - the two rows after it, as one line each, so the user can reorder.

Don't start the work and don't change anything in the tool. To start, the user runs `/project-manager:continue`.
