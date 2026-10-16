---
name: continue
description: Resumes work from a PROJECT_<name>.md queue — reads the file, checks the next ticket is still open, proposes it, and on a yes moves it to in-progress (file and tool) and starts it following the repo's own workflow. Also handles manual reorders of the queue. Run by the user as /project-manager:continue [project file name or path].
argument-hint: "[project file name or path]"
---

# project-manager:continue

Read `../../references/project-file.md` and `../../references/tools.md`. Both paths are relative to this skill's base
directory.

The repo's own rules come first. If `AGENTS.md`, `CLAUDE.md` or `CLAUDE.local.md` say how to start an issue (branch
names, working-memory folders, skills to load, who runs the tests, never commit), follow them.

## 1. Load

1. **Resolve the file** from `$ARGUMENTS`, following the contract.
2. **Check the repo**: confirm the cwd belongs to the repo listed under "Work repo(s)" in `## Source` (compare it with
   `git remote -v`). If it doesn't, say so and ask before going on, so work never lands in the wrong place.
3. **Light sync**: list the scope's statuses in one call, then:
   - delete the rows whose tickets are closed;
   - update the statuses that changed.

   Leave anything stranger than that to `/project-manager:sync`, and mention it.

## 2. Propose

1. Pick the next row following the contract, and read that ticket.
2. Show a brief: the ticket, its title, its link, its status, two or three lines about what it is, and its "Done
   when".
3. Ask with `AskUserQuestion`:
   - "Start <ticket> (Recommended)";
   - "Pick another" (list the next 3 rows);
   - "Reorder the queue";
   - "Show the full queue".

**Reorder**: when the user asks to move a row ("put #15 before #16", "move PP-9 to the top"), rewrite the rows in the
`## Queue` table, renumber them from 1, and show the changed part. The user's order wins over `## Order logic`. Leave
the rule text as it is unless the user asks to change it.

## 3. Start

1. Ask once whether to set the ticket to the tool's in-progress status, both online and in the file. On a yes, write
   it to the tool using the IDs in `## Source`, then update the row. On a no, leave both as they are.
2. Hand over to the repo's workflow: the branch naming, the working-memory folder (for example, the `local-context`
   skills if the repo uses them), and the skills the ticket names. Then begin the ticket's first step.
3. When the work reaches review (a PR is opened, or the user says so), offer to move the ticket to the in-review
   status the same way. When the ticket is closed, delete its row and renumber.

Never commit, push or merge unless the user asked for it.
