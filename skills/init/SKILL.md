---
name: init
description: Sets up a PROJECT_<name>.md work queue for a repo from a ticket tool's project scope (GitHub Project, Jira epic, GitLab milestone, Trello board, ClickUp list, …). Detects the tool from the cwd, asks for the scope and the ordering logic, then writes the ordered list of open tickets with their status. Run by the user as /project-manager:init [free text].
argument-hint: "[free text: link, tool, ordering rule, …]"
disable-model-invocation: true
---

# project-manager:init

Creates a project file: an ordered queue of the open tickets in one scope of a ticket tool, which later sessions read
to know what to do next. It can be run at the start of a new session or in the middle of one.

Before anything else, read `../../references/project-file.md` (the file contract) and `../../references/tools.md`
(how to reach each tool). Both paths are relative to this skill's base directory.

`$ARGUMENTS` is free text. Anything it already answers (a link, the tool, the ordering rule, "commit it") is not asked
again.

## 1. Detect

1. Repo root: `git rev-parse --show-toplevel`, else the cwd. Read the remote host with `git remote -v`, and look for
   the hints listed in `tools.md`.
2. Check which access routes exist: connected MCP servers for the tool (look at the tool list, loading deferred tools
   with ToolSearch), and CLIs (`command -v gh glab acli jira az`).
3. List the existing project files (see the contract).

## 2. Ask

Ask everything in **one** `AskUserQuestion` call (up to 3 questions), and skip any question `$ARGUMENTS` already
answers:

1. **Tool**: the detected one first, labelled `(Recommended)`, then the likely alternatives (for a GitHub remote:
   GitHub Projects, Jira; for Bitbucket: Jira, Trello).
2. **Scope**: what the queue covers in that tool: a GitHub project (name, number or link), a Jira epic (key or link),
   a GitLab milestone or epic, a Trello board, a ClickUp list. Offer "I'll paste the link" as an option, so the user
   types it in Other. If the tool can list its scopes (for example `projects_list` `list_projects`, or the epics of a
   Jira project), offer the most recent ones as options.
3. **Order logic**, with options:
   - "Bugs first, then by impact, features last (Recommended)": bugs by priority, then other work by priority or
     impact, then new features;
   - "By the tool's priority field or labels";
   - "Keep the tool's current order (board or rank order)";
   - Other: the user describes their own rule.

Don't ask whether to commit the file: project files are always local (see the contract, § Local only).

If the scope cannot be reached (wrong link, no access), say exactly what failed and ask again. Don't create an empty
file.

## 3. Fetch and order

1. List every item in the scope with its status (paginate fully; see `tools.md` § Reading efficiently). Drop the
   closed ones.
2. Collect the IDs `## Source` needs: the status field and option IDs, the project or board IDs, the item IDs (the
   `Ref` column).
3. Apply the order logic.
   - When it needs more than titles (impact, type, dependencies), read the labels, priority, issue type and bodies.
     Look for dependency lines such as "Start after #11", "blocked by" or "depends on", and always put a ticket after
     the tickets it depends on.
   - Trackers and parent issues go right after their last child.
   - Note any choice that wasn't obvious (an unlabelled security defect counted as a bug, a dependency that conflicts
     with priority). Report it in step 5, and don't put it in the file.

## 4. Write

- Name the file by the contract. If a file with that name already exists, ask whether to refresh it (keeping the
  user's manual order for the rows already there) or to stop.
- **Location**:
  - When this is the only project, the file goes at the repo root.
  - When a project file already exists (at the root or as a legacy `PROJECT.md`), create `.projects/` if it isn't
    there and put all project files inside it, moving existing ones with plain `mv`.
  - Then grep `CLAUDE.local.md` for the old paths, and offer to update them.
- **Ignore first**: before writing, make sure `.gitignore` holds the project-manager lines, and handle any
  tracked project file, as the contract says (§ Local only).
- Write `# PROJECT_<name>`, the version marker, `## Source`, `## Order logic` and `## Queue`, exactly as the contract
  lays them out.
- Offer, without doing it, to add a pointer to the project file in `CLAUDE.local.md`, so future sessions read it
  before picking work. Never add it to tracked docs such as `AGENTS.md` or `CLAUDE.md`.

## 5. Report

In a few lines: the file path (and that it is git-ignored locally), how many tickets it lists, and how they are grouped (for example, "4 P1 bugs, then 6
P2 bugs, …"). Add the non-obvious ordering choices as questions the user can answer to reorder. Then name the next
commands: `/project-manager:next`, `/project-manager:continue`, `/project-manager:sync`.

Never commit, push, or change ticket status in this skill unless the user asked for it in `$ARGUMENTS`.
