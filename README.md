# project-manager

A Claude Code plugin that keeps an ordered work queue for one scope of a ticket tool in a Markdown file in your repo:
a GitHub Project, a Jira epic, a GitLab milestone, a Trello board, a ClickUp list, and so on. Sessions read the file to
know which ticket comes next, keep its statuses in line with the tool, and delete a row once its ticket is closed.

## Install

Add this repository as a marketplace and install the plugin:

```
/plugin marketplace add leonardoserra/project-manager
/plugin install project-manager@project-manager
```

## The project file

Each project gets one self-describing file:

| Tool | File |
|---|---|
| GitHub Projects | `PROJECT_<project name>.md` |
| Jira (an epic) | `PROJECT_JIRA_<epic key>.md` |
| Others (GitLab, Trello, ClickUp, Linear, Azure Boards, …) | `PROJECT_<TOOL>_<project name or code>.md` |

With one project, the file sits at the repo root. With more than one, all project files move to `.projects/`.

Each file holds three sections:

- **Source**: the tool, the scope with its link, the repo(s) where the work happens, the access route (MCP server, CLI
  or API token env var), and the IDs needed to read and set the status.
- **Order logic**: your ordering rule, in plain words.
- **Queue**: the open tickets in order (`# | Ticket | Ref | Title | Status`), with each status spelled as the tool
  spells it.

Rules for the queue:

- A closed ticket's row is deleted.
- Your manual order always wins. Ask Claude to move rows, or edit the table yourself.

See [`references/project-file.md`](references/project-file.md) for the full contract, and
[`references/tools.md`](references/tools.md) for how each tool is reached.

## Skills

### `/project-manager:init [free text]`

Sets up a new project file.

1. Detects the tool from the cwd: the git remote host, plus hints such as Jira keys in the commits.
2. Asks in one go:
   - which tool to use;
   - which scope (project, epic, board or list link);
   - the ordering logic: bugs first, then impact, features last; by priority or labels; the tool's own order; or your
     own rule;
   - whether to commit the file or keep it git-ignored.
3. Fetches the open tickets, orders them (dependencies always go first), and writes the file.

Anything the free text already answers is not asked again. You can run it in a new session or partway through one.

### `/project-manager:next [project]`

Read-only. Gives a short brief of the next ticket: what it is, its "Done when", its dependencies, and the two rows
after it. If several projects exist and none is named, it asks which one.

### `/project-manager:continue [project]`

Starts work on the next ticket:

1. Checks that the cwd is the repo the file names.
2. Drops rows whose tickets were closed.
3. Proposes the next ticket.
4. On a yes, moves the ticket to in-progress (asking first before it writes to the tool) and follows the repo's own
   workflow rules.

It also handles reorders, such as "put #15 before #16".

### `/project-manager:sync [project]`

Refreshes one project file, or all of them when no project is named, from the live tool.

- Applied without asking: rows of closed tickets are deleted; forward status changes and title changes are updated.
- Asked in one question per file: backward status moves, unknown statuses, new or reopened tickets, tickets moved out
  of the scope, and tickets that can't be found.
- It never reorders rows and never writes to the tool.

### `/project-manager:cleanup <project>`

Checks in the tool that every listed ticket is really closed, then deletes the file. If some tickets are still open,
it lists them and asks whether to keep and sync the file, delete it anyway, or cancel.

## Layout

- `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`: the plugin and marketplace manifests.
- `references/project-file.md`: the project file contract (name, location, sections, picking the next ticket).
- `references/tools.md`: tool detection and access per tool (GitHub, Jira, GitLab, Trello, ClickUp, Linear, Azure
  Boards).
- `skills/<init|next|continue|sync|cleanup>/SKILL.md`: the skills.

## License

[MIT](LICENSE)
