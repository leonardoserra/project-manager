# Changelog

All notable changes to this project are documented in this file.

## 0.1.1 - 2026-10-16

- Changed
  - Project files are now local by default. The skills add `PROJECT_*.md`, `PROJECT.md` and `.projects/` to
    `.gitignore` and never commit it, and `/project-manager:init` no longer asks whether to commit the file. To share a
    project file anyway, `git add -f` it or drop the lines from `.gitignore`.
  - A project file that is already tracked is flagged once, with an offer to untrack it while keeping the local copy.
    If you keep it tracked, the skills keep working on it.
  - Pointers to a project file now go in `CLAUDE.local.md` instead of `AGENTS.md` or `CLAUDE.md`.

## 0.1.0 - 2026-10-16

- Changed
  - New `/project-manager:init [free text]` creates a work queue file for one scope of a ticket tool: a GitHub
    Project, a Jira epic, a GitLab milestone, a Trello board, a ClickUp list, and others. It detects the tool from the
    repo's remote and asks, in one go, for the scope, the ordering rule, and whether to commit the file.
  - The queue is named `PROJECT_<project>.md` for GitHub, `PROJECT_JIRA_<epic>.md` for a Jira epic, and
    `PROJECT_<TOOL>_<name>.md` for other tools. Once a repo follows more than one project, all queue files move to
    `.projects/`.
  - Each file records where its tickets live (tool, scope link, work repo, access route, status IDs), the ordering
    rule, and the open tickets in order with their status.
  - New `/project-manager:next [project]` gives a short brief of the next ticket without starting it.
  - New `/project-manager:continue [project]` proposes the next ticket, moves it to in-progress after asking, and
    follows the repo's own workflow. It also reorders the queue on request.
  - New `/project-manager:sync [project]` updates one queue, or all of them, from the live tool: rows of closed
    tickets are dropped and changed statuses are updated. Backward moves, new or reopened tickets, and tickets moved
    out of scope are asked about first.
  - New `/project-manager:cleanup <project>` deletes a finished queue once the tool confirms every ticket is closed.
    If some are still open, it asks first.
