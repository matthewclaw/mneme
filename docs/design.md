# Design: Storage, Schemas and Repository Matching

Part of the [implementation plan](implementation-plan.md). Status: proposed.

## Storage layout

```
%LOCALAPPDATA%\Mneme\          machine-specific, never synced
  config.json
  registry.json
%USERPROFILE%\Mneme\           knowledge, can be synced later
  projects\<project-id>\
    manifest.json
    project-overview.md
    backlog.md
    open-questions.md
    requirements\  architecture\  decisions\  meetings\  risks\  planning\
```

- `risks\` also holds assumptions.
- On other platforms, the state folder is `$XDG_STATE_HOME/mneme` or `~/.local/state/mneme`, kept to a single function. The knowledge root comes from `os.homedir()` on every platform.
- **Keep the knowledge root out of OneDrive and similar sync folders.** A work OneDrive may belong to one client's tenant, and syncing there could put other clients' material in it. `doctor` warns if `contextRoot` is under one.

## Schemas

All examples use synthetic data.

### config.json
```json
{ "schemaVersion": 1, "contextRoot": "C:\\Users\\<you>\\Mneme" }
```

### registry.json
An array rather than a map, so duplicates remain detectable after normalisation.
```json
{
  "schemaVersion": 1,
  "repositories": [
    {
      "gitCommonDir": "C:\\repos\\acme\\.git",
      "projectId": "acme-payments-3f9a",
      "repositoryName": "Payments API",
      "lastKnownRoot": "C:\\repos\\acme",
      "originUrl": "https://github.com/acme/payments.git",
      "registeredAt": "2026-11-02T10:00:00Z"
    }
  ]
}
```

| Field | Use |
|---|---|
| `gitCommonDir` | The identity key |
| `lastKnownRoot` | Display and `doctor` only, never matching |
| `originUrl` | A hint when attaching a moved repository, with credentials stripped. Never used to match at session start |

### manifest.json
Portable, with no machine paths.
```json
{
  "schemaVersion": 1,
  "projectId": "acme-payments-3f9a",
  "clientName": "Acme",
  "projectName": "Payments",
  "createdAt": "2026-11-02T10:00:00Z"
}
```

### Rules

- **Project ID:** a `<client>-<project>` slug plus 4 random hex characters. It is never recomputed, so renames don't break it. Pattern: `^[a-z0-9][a-z0-9-]{0,62}$`.
- **Versioning:** a `schemaVersion` above the supported one returns `invalid`. Migration code waits for a schema v2.
- **Writes:** temp file then atomic rename, keeping `registry.json.bak`. The hook only reads.

## Repository matching

Identity is the normalised output of:

```
git rev-parse --path-format=absolute --git-common-dir
```

This was checked on Windows with Git 2.55: a linked worktree reports the main repository's `.git`, and a folder outside any repository exits 128. `--path-format=absolute` is required, because the main checkout otherwise returns a relative `.git`.

**Normalisation:** `path.resolve`, no trailing separator, and a case-insensitive comparison on Windows. Other platforms compare exactly.

| Case | Behaviour |
|---|---|
| Normal repository, any subfolder | Git finds the root; exact match |
| Linked worktree | Same common directory, same project |
| Nested repository or submodule | Innermost repository only. If unregistered there is no context; it **never inherits the parent's** (to be confirmed by an M1 test) |
| Second clone | Different identity, unregistered until attached with `mneme register --project <id>` |
| Moved repository | No match, so no context. `doctor` lists entries whose path is gone. `register` offers to attach when `originUrl` matches, and the user confirms |
| Same origin, new path | **Never matched automatically at session start** |
| Duplicate entries for one identity | `ambiguous`: no context, diagnostic shown |
| Manifest missing, invalid or wrong ID | `invalid`: no context, diagnostic shown |
| 8.3 names, `subst`, mapped drives | May not match, which means no context and never the wrong project |

**Rule:** every normalisation miss falls back to "not registered", never to a guess. There is no folder-name or prefix matching.

### Git usage

- Only `rev-parse` and config reads are used, with `GIT_OPTIONAL_LOCKS=0` and with `GIT_DIR` and `GIT_WORK_TREE` cleared.
- `git status` is never used, because it can refresh the index and run repository-configured `fsmonitor` commands.
- Git runs through `execFile` with no shell, an absolute path and a 5 s timeout. See [Security review](security.md) for why the path is absolute.
- "Dubious ownership" errors become a diagnostic.
