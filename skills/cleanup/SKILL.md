---
name: cleanup
description: Retires a finished PROJECT_<name>.md — verifies in the live tool (GitHub, Jira, Trello, …) that every listed ticket is really closed, then deletes the file; if some are still open, lists them and asks before deleting. Run by the user as /project-manager:cleanup <project file name or path>.
argument-hint: "<project file name or path>"
disable-model-invocation: true
---

# project-manager:cleanup

Read `../../references/project-file.md` and `../../references/tools.md`. Both paths are relative to this skill's base
directory.

1. **Resolve the file** from `$ARGUMENTS`, following the contract. If no argument is given, ask which file, even when
   only one exists, because the file is about to be deleted.
2. **Check every row** in the tool from `## Source`, using one listing of the scope plus direct lookups for any ticket
   missing from it. Each ticket is either closed (as `## Source` defines closed for that tool), still open, or not
   found.
3. Also note any open tickets in the scope that are missing from the file. They don't block the cleanup, but the user
   should know about them.
4. **Decide**:
   - **Everything closed**, and no open ticket missing from the file: delete the file.
   - **Otherwise**: list the open tickets, the not-found tickets and the missing ones, each with its key, title and
     status. Then ask with `AskUserQuestion`:
     - "Keep the file and sync it (Recommended)", which runs the `sync` rules on it;
     - "Delete it anyway";
     - "Cancel".
5. **Delete**:
   - Remove the file with `git rm` if it is tracked, plain `rm` otherwise. Don't commit.
   - If `.projects/` is now empty, remove it.
   - Grep `AGENTS.md`, `CLAUDE.md` and `README*` for the file's path, and offer to remove those references (only remove
     them after a yes).
6. **Report**: one line with what was checked and whether the file was deleted. List any open tickets that were left
   behind.
