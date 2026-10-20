# Project file contract

Every project-manager skill reads and writes files in this shape. The file describes itself: its `## Source` section
holds everything a later session needs to find the tickets again, so no skill depends on memory of the session that
created it.

## Name and location

| Tool | File name |
|---|---|
| GitHub Projects | `PROJECT_<project name>.md` |
| Jira (epic as scope) | `PROJECT_JIRA_<epic key>.md` |
| Any other tool (GitLab, Trello, ClickUp, Linear, Azure Boards, …) | `PROJECT_<TOOL>_<project name or code>.md` |

- `<TOOL>` is upper case (`GITLAB`, `TRELLO`, `CLICKUP`, `LINEAR`, `AZURE`).
- Names are sanitized: keep letters, digits, `-` and `_`; spaces and other characters become `-`. For example, the
  GitHub project "Pin Party" becomes `PROJECT_Pin-Party.md`, and the Jira epic `PP-42` becomes `PROJECT_JIRA_PP-42.md`.
- **One project** in the repo: the file sits at the repo root.
- **Two or more**: all project files live in `.projects/` at the repo root (moved with plain `mv`).
- Finding the files: `ls PROJECT_*.md .projects/PROJECT_*.md 2>/dev/null` from the repo root (`git rev-parse
  --show-toplevel`, else cwd). A legacy unprefixed `PROJECT.md` also counts. Offer to rename it to the convention, but
  only rename after the user says yes.

## Local only

Project files are a personal tool, not part of the repo by default. They are always git-ignored.

- **Ignore them in the repo-root `.gitignore`** (create it if missing). It must hold these lines; append the missing
  ones, and never duplicate a line:

  ```
  # project-manager (local work queues)
  /PROJECT_*.md
  /PROJECT.md
  /.projects/
  ```

- **Every skill that writes a project file** checks these lines first and adds the missing ones, without asking.
  Never stage or commit the `.gitignore` change: whether to push it is the user's call.
- **Pushing a project file anyway** is the user's choice: they can `git add -f` it, or remove the lines from
  `.gitignore`. Never do either on your own, and never re-add the lines to a `.gitignore` the user changed on purpose
  (a removed block in the git history, or a tracked project file the user says to keep).
- **A tracked project file** (`git ls-files PROJECT_*.md PROJECT.md .projects/`): the user may have pushed it on
  purpose. Mention it once, and offer to untrack it with `git rm --cached <file>`, which keeps the local copy. Only run
  it after a yes, never commit, and if the user wants to keep it tracked, keep working on it as is.
- **Never add a reference to a project file to tracked files** (`AGENTS.md`, `CLAUDE.md`, `README*`, …). A pointer
  for future sessions goes in `CLAUDE.local.md`, and only when that file is git-ignored too (`git check-ignore -q
  CLAUDE.local.md`); otherwise add `/CLAUDE.local.md` to `.gitignore` first, after asking. References the repo's
  tracked docs already hold are the repo's choice: follow them and leave them as they are.
- Outside a git repo, there is nothing to ignore: skip these steps.

## Resolving the `<project>` argument

- **A path**, or **a name that matches a file exactly**: use that file.
- **A partial name** (`pinparty`, `PP-42`, `trello`): match it case-insensitively against the file names. One match:
  use it and say which file it is. Several matches: ask with `AskUserQuestion`.
- **No argument** and exactly one file: use it. No argument and several files: ask (most recently modified first,
  with the full list printed in the question text).
- **No project file**: say so, suggest `/project-manager:init`, and stop.

## Layout

```markdown
# PROJECT_<name>

<!-- project-manager: 1 -->

## Source

- Tool: GitHub Projects
- Scope: project "PinParty" #3, https://github.com/orgs/GaleonWare/projects/3
- Work repo(s): GaleonWare/PinParty (git@github.com:GaleonWare/PinParty.git)
- Access: GitHub MCP `projects_list` (owner `GaleonWare`, owner_type `org`, project_number `3`); fallback `gh`
- Status field: `Status` id `427523340`; options Backlog `f75ad846`, Ready `61e4505c`, In progress `47fc9ee4`,
  In review `df73e18b`, Done `98236657`
- Statuses, in workflow order: Backlog → Ready → In progress → In review → Done
- Closed means: the issue's state is `closed`

## Order logic

Bugs first, by impact (P1 → P3); then other work by impact; new features last. A ticket that depends on another goes
after it.
Manual moves win: the queue order is authoritative, and sync never reorders existing rows.

## Queue

| # | Ticket | Ref | Title | Status |
|---|---|---|---|---|
| 1 | #22 | 273701911 | verify.sh can pass without running DB tests | Backlog |
```

### Section rules

- **`## Source`** has everything needed to reach the tickets: the tool, the scope with a link, the repo(s) where the
  code work happens, the access route (MCP server/tool, CLI, or API with the name of its token env var, never the
  token itself), the IDs needed to read and set the status, the tool's status names in workflow order, and what
  "closed" means in that tool (see `tools.md`).
- **`## Order logic`** states the user's ordering rule in one or two plain sentences. When the user later moves a row,
  keep the rule as it is: the queue order is what counts.
- **`## Queue`**:
  - `#` is the position, renumbered from 1 after every insert, move or delete.
  - `Ticket` is the human key (`#22`, `PP-42`, the Trello card short link).
  - `Ref` is the ID that makes the fastest API call (the project item ID, Jira issue ID, card ID, ClickUp task ID). Use
    `-` when the tool needs only the key.
  - `Status` uses the tool's own status name, exactly as the tool spells it.
- A row is deleted when its ticket is closed. The file holds only open work.
- Never store credentials, tokens or private comments in the file.

## Picking "next"

1. First, a row whose status is an in-progress kind (In progress, In development, Doing), because that work is
   unfinished.
2. Otherwise, the first row whose status is not In review, Done or its equivalents (QA, Resolved, Closed).
3. If the next row names a dependency that is still open in the file (for example "Start after #11" in its body),
   say so and propose the dependency instead.
