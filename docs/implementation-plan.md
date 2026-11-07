# Mneme MVP: Implementation Plan

> **Superseded, do not build from this yet.** The project's scope has grown from single-user session context to portable project knowledge, and planning has reopened. This plan stays as a reference until it is rewritten. Current decisions and open questions are in [decisions.md](decisions.md).

**Status:** planning reopened, implementation not started. First planned on 2026-11-02 from [project-brief.md](../project-brief.md).

Supporting documents:
- [Design: storage, schemas and repository matching](design.md)
- [Claude Code integration findings](claude-code-integration.md)
- [Security review](security.md)

## Purpose

Client repositories may not allow AI configuration or extra tracked files, but project knowledge still needs to persist across Claude Code sessions. Mneme keeps that knowledge outside the repository, maps each repository to its project, and tells Claude at session start where the knowledge lives.

It is a small integration utility, not an agent framework.

## Decisions that differ from the brief

| Brief | Decision | Why |
|---|---|---|
| Manifest holds the context location | Location is derived as `contextRoot\projects\<id>` | An absolute path in a syncable file would go stale |
| `templates\` in the data folder | Templates ship with the tool | A user-editable template folder can wait until someone wants one |
| No home for outstanding questions | Added `open-questions.md` | The brief requires it, but its layout omitted it |

Other decisions:

- **Language:** plain JavaScript (ES modules) with `// @ts-check` and JSDoc types, with no dependencies and no build step. Node 26 can run `.ts` files directly during development, but it won't strip types from packages under `node_modules`, so a published TypeScript package would still need a compiler and a build. Moving to TypeScript later is mechanical.
- **Read access:** Claude prompts when reading context, with no allow rules. See [Security review](security.md).
- **No hook installer in the MVP:** the README gives a snippet to paste, and `mneme doctor` checks it and prints the correct version for the machine. For public distribution, a Claude Code plugin is the better route, because plugins carry their own `hooks/hooks.json`.
- **Auto memory is not replaced:** Claude Code's auto memory is already per-repository, outside the repository and shared across worktrees. Mneme is justified by curated human-owned documents, many-repositories-to-one-project mapping, and not being tied to Claude. It should not grow into note capture.

## Requirements

**Explicit**
- Windows, Claude Code first, local filesystem only.
- No database, service, web server, vector database, external API, telemetry or orchestration. No ECC dependency.
- Never modify the client repository.
- Commands: `register`, `list`, `inspect`, `context`, `unregister`, and `doctor` if appropriate.
- The hook gives a concise bootstrap only and never blocks a session.
- Ambiguous or failed resolution never selects a project.
- The core has no Claude dependency. Machine-specific paths stay separate from knowledge.

**Assumptions**
- Single user on one machine. Sync stays a future option.
- Git and Node are installed. A folder that is not a Git repository is treated as unregistered.
- One project can have many checkouts, but each checkout belongs to one project.
- Knowledge is edited by hand, or by Claude with the user's approval.
- Code stays portable (`node:path`, `node:os`), with one platform branch for the state folder and path comparison. Only Windows is tested and supported in the MVP.

## Architecture

| File | Responsibility | Depends on |
|---|---|---|
| `bin/mneme.js` | Single entry point, dispatches commands | modules below |
| `src/core.js` | Registry, manifest, path validation, resolver | git |
| `src/cli.js` | Prompts (`node:readline/promises`) and management commands | core |
| `src/claude-adapter.js` | Hook stdin to resolver to hook JSON | core |
| `templates/` | Starting context files, copied at registration | none |
| `package.json` | `bin` entry, `engines`, no dependencies, `"private": true` until published | none |

Registry, manifest and resolver share `core.js` until size justifies a split. A future Codex adapter would be one new file. `npm link` puts `mneme` on PATH during development.

```
SessionStart -> node bin/mneme.js hook claude
  stdin JSON (cwd) -> resolveProject(cwd) -> { status, project?, diagnostic? }
    registered                  -> additionalContext (bootstrap text)
    not-a-repo / unregistered   -> no output, exit 0
    ambiguous / invalid / error -> systemMessage to the user, no context, exit 0
```

## Runtime

Node 26 or later (`engines: >=26`), pinned for development in `.nvmrc`, so `nvm use` selects it.

- **The hook must not depend on the active Node version.** Client projects may need a different Node, and `.nvmrc` only applies inside this repository. The hook therefore uses the absolute path of a Node 26 install, not a path that follows `nvm use`.
- **The hook checks its own Node version.** On an older Node it reports a diagnostic and exits 0, and `doctor` flags it.
- **Upgrading or removing that Node install breaks the path.** The result is a non-blocking hook error and a normal session, and `doctor` prints the updated snippet.
- Claude Code's native installer doesn't guarantee Node, so a user without it gets a non-blocking hook error and a normal session. The README lists Node as a prerequisite.
- The npm name `mneme` is taken by an unrelated package. The package would be `@matthewclaw/mneme`, with the command still `mneme`. This only matters when publishing.

## Milestones

Each milestone is small and leaves its tests passing.

### M1: Core resolver
Files: `package.json`, `src/core.js`, `test/core.test.js`

- **Accept:** every case in [Repository matching](design.md#repository-matching) returns the expected status. Malformed JSON and future schema versions return `invalid` without throwing.
- **Tests:** normal repository, subfolder, worktree, submodule, nested repository, bare-repository worktree, case differences, duplicates, traversal ID, junction, non-ASCII path, planted `git.exe`.

### M2: CLI
Files: `bin/mneme.js`, `src/cli.js`, `templates/`, `test/cli.test.js`

- `register`: detect root, check whether already registered, prompt for three names, generate ID, suggest path, confirm, create files, then write the registry. `--project <id>` attaches a checkout to an existing project.
- `list`, `inspect` (resolved status and validity), `context` (prints the path only, for scripting), `unregister` (removes the mapping only).
- **Accept:** the client repository is unchanged after every command. `unregister` keeps all context files. Re-registering creates nothing. Declining confirmation writes nothing.

### M3: Claude adapter
Files: `src/claude-adapter.js`, `test/hook.test.js`

- **Accept:** for every status, piped JSON yields valid output and exit 0. Output parses with exactly the expected fields. The bootstrap is under 1,000 characters. Empty stdin, garbage stdin, a missing `cwd` and missing Git all exit 0.

### M4: Live verification with Claude Code
No new code expected. This edits `~/.claude/settings.json`, so it needs explicit approval first.

- Confirm on the installed version:
  - context arrives
  - `args` works
  - a marker `node.exe` in the test repository isn't run
  - `systemMessage` is visible
  - `startup`, `resume`, `clear` and `compact` all restore context
  - a corrupted registry doesn't block a session
  - unregistered repositories see nothing
  - typical hook time is under 1 s
- Check the VS Code extension too, since it may bundle a different version from the CLI.
- Record the results in the README. If something fails, fix the adapter only.

### M5: `doctor` and README
- `doctor` is read-only and checks: Node version, the hook's Node path, Git present, config and registry parse, every manifest valid, stale paths, hook present in settings, `contextRoot` inside the profile and outside any OneDrive folder, and any broad `Read` allow rule.
- The README covers prerequisites, install, hook snippet, data-handling note, repository-write warning and `/add-dir` warning.
- **Accept:** each `doctor` warning has a test that triggers it.

## Testing strategy

- **Runner:** Node's built-in `node:test` and `node:assert`, run with `npm test` (`node --test test/`). No install needed.
- **Isolation:** each test uses a temp folder and runs Mneme in a child process with `LOCALAPPDATA` and `USERPROFILE` pointed there. Real data can't be touched, and production code needs no test switches.
- **Unit:** normalisation, slug and ID generation, schema validation, name sanitising, credential stripping.
- **Integration:** real Git repositories, worktrees and submodules in temp folders, with the hook driven through stdin and stdout.
- **Repository-unchanged check:** snapshot file hashes and `git status --porcelain` around register, inspect and hook runs.
- **Data:** synthetic only (`Acme`, `Payments`), never real client data.
- **Versions:** Node 26. CI across Node versions and operating systems can wait until the project is public.
- **Manual:** M4 only, since it depends on Claude Code itself.

## End-to-end verification

1. `npm test` passes on Node 26.
2. In a synthetic temp repository, run `mneme register`, start Claude Code and ask "what project is this?". It answers from the bootstrap, and reading `project-overview.md` prompts for permission.
3. From a linked worktree, the same project resolves. From an unregistered repository, nothing is added.
4. With a corrupted `registry.json`, the session still starts and shows a diagnostic.
5. `git status` in the test repository stays clean throughout.
6. After `mneme unregister`, the mapping is gone and the context files remain.

## Open questions

This doesn't block M1, and the default applies unless changed.

**Should each client get a separate context root?** Default: one root, separated by project folder. If a client's policy requires physical separation, `contextRoot` could become per-project, which is a small schema change that is cheaper before the first release.

## Out of scope

- `project-context` skill, automatic knowledge updates, transcript capture.
- Scoped PreToolUse read hook. This is the natural follow-up if read prompts become annoying.
- `CwdChanged` handling for `/cd`.
- Sync, Codex adapter, other version control systems.
- Testing or supporting macOS and Linux, npm publishing, plugin packaging, CI. The code stays portable so these remain cheap.
- Semantic search, indexing, LLM lookups.
- Automatic editing of `settings.json`, schema migrations, user-editable templates.
- Automatic relinking of moved repositories.
