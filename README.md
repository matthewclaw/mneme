# Mneme

Project knowledge for AI coding sessions, kept outside the repository.

> **Status: planning.** No code exists yet, so the commands below don't work. This README describes the intended MVP.

## Why

Each Claude Code session starts fresh. The usual fix is to commit a `CLAUDE.md` or other AI config to the repository, but client repositories often don't allow that: no AI-specific files, no extra tracked files, no `.gitignore` changes.

Mneme keeps project knowledge in a folder of its own and connects it to the repository from the outside, so future sessions can pick up the project's context while the repository stays untouched.

## How it works

1. **Register a repository once.** From inside the repository, `mneme register` asks for the client, project and repository names, then creates a context folder for the project and records which repository it belongs to. Nothing is written to the repository.
2. **Keep knowledge in the context folder.** It holds ordinary Markdown files: overview, requirements, architecture, decisions, meetings, risks, planning, backlog and open questions.
3. **Start Claude Code as usual.** A user-level `SessionStart` hook checks whether the repository is registered. If it is, Claude is told which project this is and where its context lives.
4. **Claude reads what it needs.** Documents aren't loaded into every session. Claude opens the relevant ones when they're useful, and you approve those reads.

Unregistered repositories are unaffected. If anything goes wrong, you get a short diagnostic and the session carries on normally without project context.

## Commands

| Command | What it does |
|---|---|
| `mneme register` | Links the current repository to a new or existing project |
| `mneme list` | Lists registered repositories |
| `mneme inspect` | Shows which project the current repository resolves to, and whether it's valid |
| `mneme context` | Prints the current project's context folder |
| `mneme unregister` | Removes the link and keeps the context files |
| `mneme doctor` | Checks the setup and reports problems without changing anything |

## Where things live

```
%LOCALAPPDATA%\Mneme\          this machine only: config and repository links
%USERPROFILE%\Mneme\projects\  project knowledge, one folder per project
```

Worktrees of a repository share its project, and several clones can be linked to one project. If a repository can't be matched with certainty, Mneme gives no context rather than risk showing another project's.

## What it doesn't do

- **It isn't an agent framework.** There's no orchestration, and it doesn't replace Claude Code features such as auto memory.
- **It doesn't update knowledge on its own.** You write the documents, or Claude proposes changes for you to approve.
- **It doesn't use any services.** There's no database, background process, search index, telemetry or network call.

## Client data

Mneme stores files locally, but anything Claude reads from them is sent to the model provider and kept in local session transcripts. Keeping context outside the repository doesn't by itself make it permissible to use, so each client's data-handling policy still applies.

Keep the context folder out of OneDrive and similar sync folders, especially ones that belong to a client's tenant.

## Requirements

Planned for Windows, Claude Code, Git and Node 26 or later, with no other dependencies. The repository includes an `.nvmrc` for development. The code is intended to stay portable, but other platforms won't be tested at first.

## More detail

- [Implementation plan](docs/implementation-plan.md)
- [Design: storage, schemas and repository matching](docs/design.md)
- [Claude Code integration findings](docs/claude-code-integration.md)
- [Security review](docs/security.md)
- [Original brief](project-brief.md)

## License

[MIT](LICENSE)
