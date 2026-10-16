# Tool access

Try the access routes in this order: a connected **MCP server** for the tool, then its **CLI** if it is installed
(`command -v`), then the **REST API** using a token from an env var the user names. If none works, tell the user what
to set up (connect the MCP server, `! gh auth login`, export a token) and stop. Don't guess an answer.

Reads are free to do. **Writes** (changing a ticket's status, moving an item) publish to a shared tool, so ask before
each one unless the user already said yes in this session for that kind of change.

## Detecting the tool from the cwd

- Run `git remote -v` and read the host: `github.com` means GitHub, `gitlab.*` means GitLab, `bitbucket.org` means
  Bitbucket (whose tickets usually live in Jira), and Azure DevOps (`dev.azure.com` / `visualstudio.com`) means Azure
  Boards.
- Hints for Jira: commit subjects or branch names that hold keys such as `ABC-123`
  (`git log --oneline -50 | grep -E '[A-Z][A-Z0-9]+-[0-9]+'`), or a `.jira*` config file.
- Hints for Trello or ClickUp: links in `README*`, `CONTRIBUTING*`, `AGENTS.md` or `CLAUDE.md`.
- The guess is only a suggestion. Always confirm it with the user, because the tickets may live in a tool other than
  the code host.

## Per tool

| Tool | Scope unit | Read the items | Closed means | Status lives in |
|---|---|---|---|---|
| GitHub Projects | project (org or user, number) | MCP `projects_list` method `list_project_items` with `field_names: ["Status"]` (paginate); `gh project item-list <n> --owner <o> --format json` | issue `state: closed` (a PR: merged or closed) | project `Status` single-select field |
| Jira | epic (or a JQL filter, sprint or board if the user prefers) | MCP Atlassian `searchJiraIssuesUsingJql` with `parent = <EPIC>` (classic projects: `"Epic Link" = <EPIC>`); `acli`/`jira` CLI; REST `/rest/api/3/search?jql=` | `statusCategory = Done` | workflow status, with its category |
| GitLab | epic, milestone or iteration (or a board label) | `glab issue list --milestone`/`--epic`, or REST `/projects/:id/issues?milestone=` / `/groups/:id/epics/:iid/issues` | `state: closed` | board list labels (scoped `status::*` labels if used), else opened/closed |
| Trello | board (optionally one list) | MCP if connected; REST `/1/boards/<id>/cards?fields=name,idList,closed,shortLink` + `/1/boards/<id>/lists` (`TRELLO_KEY`/`TRELLO_TOKEN`) | card `closed: true`, or in the list the user names as done | the card's list |
| ClickUp | list, folder or space | MCP if connected; REST `/api/v2/list/<id>/task?include_closed=true` (`CLICKUP_TOKEN`) | status `type: closed` (or `done`) | task status |
| Linear | project or cycle | MCP Linear; GraphQL `issues(filter: {project: …})` | state type `completed` or `canceled` | workflow state |
| Azure Boards | area path, iteration or query | `az boards query --wiql`; REST WIQL | state category Completed | work item state |

For any other tool, ask the user how the tickets can be reached. Record that route in `## Source` and follow the same
rules.

## Reading efficiently

- Fetch the whole scope in one listing (paginated), not one call per ticket, and ask for only the fields needed
  (status, state, title, key, ID).
- A ticket in the file that is missing from the listing (for example, removed from the project or moved to another
  epic) gets one direct lookup by its key, to tell "closed elsewhere" apart from "moved" and "deleted".
- Read ticket bodies only when needed (`next`, `continue`, or ordering during `init`), and only for the tickets
  involved.
