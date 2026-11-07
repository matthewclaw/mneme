# Security Review

Part of the [implementation plan](implementation-plan.md). Status: proposed. Mitigations are design commitments until their tests exist.

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| Mneme writes to the client repository | The hook and resolver have no write paths. `register` refuses if the context target is inside the repository. A test compares the repository's tree and status before and after |
| **Claude Code itself writes to the repository** | "Yes, and don't ask again" on a Bash prompt saves to `.claude/settings.local.json` in the repository. Mneme can't prevent this, so the README warns about it. Declining the persistent option avoids it |
| **Planted executable in a client repository** | Believed behaviour, to be confirmed by test: Node on Windows checks the current folder before PATH when spawning a bare name, so a `git.exe` committed to a repository could run when the hook starts. Mneme resolves `git` to an absolute path from absolute PATH entries only. A test plants a harmless marker `git.exe`. M4 repeats this for Claude Code launching `node`, which is why the hook snippet uses an absolute path |
| Wrong project's context | Exact identity only, with ambiguity failing closed. The manifest ID must match both the folder and the registry |
| Reads across client projects | No allow rules, so each read outside the repository prompts and shows the path. **Don't add `Read(~/Mneme/**)` to settings**, because any session could then silently read any client's context. `doctor` warns if it finds one |
| `/add-dir` side effect | It removes read prompts for the session, but edits there then follow the permission mode, so `acceptEdits` would write without asking. The README warns about this |
| Path traversal or junctions | ID pattern check, the resolved path must stay under `contextRoot`, and symlinks and junctions in the project folder are rejected |
| Injection via manifest fields | Names have control characters stripped and are capped at 80 characters before going into context. File contents are never injected |
| Injection via context documents | The bootstrap labels them as reference material, not instructions. **This lowers the risk but can't eliminate it**, since Claude still reads the text. Mneme itself never executes anything from them |
| Silent knowledge rewrites | No Edit allow rules, so Claude's edits there prompt under default permissions. Mneme has no write path to existing documents |
| **Data leaves the machine via Claude** | "Local" describes storage only. Whatever Claude reads goes to the model provider and into local transcripts. Each client's data-handling policy still applies, and the README states this |
| Credentials in remote URLs | Userinfo is stripped before storing `originUrl` |
| Sensitive logs | No log file. Diagnostics contain status and paths only, never document content |
| A broken workspace blocks work | Top-level catch, always exit 0, never exit 2, 10 s hook timeout |
| Local file privacy | Both roots sit under the user profile, which is private by default on Windows, so custom ACLs aren't needed. `doctor` warns if `contextRoot` is outside the profile |

## Read access decision

Claude Code prompts for reads outside the working directories by default. The options considered:

| Option | Result |
|---|---|
| **Prompt on read (chosen)** | No settings change. The user sees each path and approves it, or runs `/add-dir` for the session |
| Scoped PreToolUse hook | No prompts and enforced isolation, but a process launch on each Read, Grep and Glob call and more code to test. The natural follow-up if prompts become annoying |
| User-level allow rule | No prompts and no code, but it breaks isolation between clients, including when prompt injection in a client repository steers a session |

## What this doesn't protect against

- Malicious software already running as the same user.
- A user approving a read prompt for the wrong project's files.
- Whatever a client's policy says about using AI tools with their information. That decision remains with the user.
