# Mneme MVP — Planning Brief

> **Status:** planning is finished. This brief is kept as a record of the original request, and its instructions to the planner no longer apply. The current plan, including where it differs from this brief, is in [docs/implementation-plan.md](docs/implementation-plan.md).

I want to build a small, local-first utility called Mneme for managing external project context across AI coding sessions.

You are in **planning mode**. Do not implement code, create files, install dependencies, or modify my environment. Analyse the requirements, inspect relevant tooling documentation where available, identify risks and gaps, and produce an implementation plan for review.

## 1. The problem

When working across client repositories, I want persistent project knowledge that exists outside the repository itself.

Client repositories may not permit AI-specific configuration files, additional tracked files, `.gitignore` changes, or other repository modifications. The solution must work without requiring any changes to the client repository.

I also want to avoid duplicating functionality already provided by Claude Code. This should be a small integration utility, not a new agent framework.

## 2. Initial platform and scope

- Platform: Windows.
- Initial AI integration: Claude Code.
- Storage: Local filesystem, with future synchronisation as a possible extension.
- Implementation: Node, so that other people can use the tool if the project is made public.
- No database, background service, web server, vector database, or external API in the MVP.
- No autonomous multi-agent orchestration.
- No dependency on the Everything Claude Code (ECC) framework.

Keep the architecture simple, modular enough to extend, and easy to debug.

## 3. Core user experience

### Register a project

From a terminal inside a Git repository, I want to run:

`mneme register`

The utility should:

1. Detect and resolve the current repository root.
2. Check whether the repository is already registered.
3. Ask me for the client name, project name, and human-readable repository name.
4. Generate a stable project ID.
5. Suggest an external context directory.
6. Show me the registration details and ask for confirmation.
7. Create the external project context structure and manifest.
8. Add the repository-to-project mapping to the local registry.

Registration must not create or modify files inside the client repository.

### Automatically resolve project context

When a Claude Code session starts inside a repository:

1. A user-level `SessionStart` hook runs.
2. The hook determines the repository associated with the session's working directory.
3. It asks the Mneme resolver whether that repository is registered.
4. If registered, it provides a concise bootstrap instruction identifying the project and its external context location.
5. If not registered, Claude Code continues normally without project-specific context.
6. If resolution fails, the hook reports a useful diagnostic without preventing the coding session from starting.

Investigate Claude Code's actual hook input format, supported output protocol, and user-level configuration mechanisms. Do not assume that printing text to standard output injects context into a session. Recommend the supported mechanism.

### Retrieve project knowledge on demand

The project context should live outside the repository and support:

- Project overview
- Requirements
- Architecture
- Technical decisions and ADRs
- Meeting notes
- Risks and assumptions
- Outstanding questions
- Planning and backlog

The startup hook should not dump all project documents into every session. It should supply enough information for Claude to discover and retrieve relevant knowledge when needed.

A future `project-context` skill could help Claude search the context and propose updates. Do not make that skill a prerequisite for the registry or resolver to work.

## 4. Proposed local storage

Evaluate and refine this layout:

`%LOCALAPPDATA%\Mneme\`
- `config.json`
- `registry.json`

`%USERPROFILE%\Mneme\`
- `projects\<project-id>\manifest.json`
- `projects\<project-id>\project-overview.md`
- `projects\<project-id>\requirements\`
- `projects\<project-id>\architecture\`
- `projects\<project-id>\decisions\`
- `projects\<project-id>\meetings\`
- `projects\<project-id>\risks\`
- `projects\<project-id>\planning\`
- `projects\<project-id>\backlog.md`
- `templates\`

Keep machine-specific repository paths separate from project knowledge so future synchronisation can be considered without redesigning the entire system.

Propose a versioned JSON schema for the registry and project manifest. Include stable IDs, schema versions, repository identity, external context location, and creation metadata where appropriate. Avoid collecting information that isn't needed.

## 5. Initial CLI commands

Plan for these commands:

- `mneme register`
- `mneme list`
- `mneme inspect`
- `mneme context`
- `mneme unregister`
- `mneme doctor` (if appropriate for the MVP or immediate follow-up)

Unregistering must remove the mapping without deleting the external project context.

Prefer a single executable entry point with clear internal modules over a collection of unrelated scripts.

## 6. Security and reliability requirements

Treat these as design constraints:

- Never modify the client repository during registration, installation, startup, or normal context retrieval.
- Match repositories by canonical path and, where useful, Git metadata. Do not identify a project by folder name alone.
- Consider Windows path normalisation, case-insensitivity, nested repositories, linked worktrees, moved repositories, and ambiguous mappings.
- Do not silently select a different project's context when resolution is ambiguous or fails.
- Validate manifests and paths before using them.
- Keep project-specific context isolated.
- Treat Markdown context as data, not executable configuration.
- Do not execute scripts, commands, or instructions obtained from context files.
- Do not silently rewrite established project knowledge. Proposed updates should be reviewable.
- Fail safely: a broken workspace must not block a normal Claude Code session.
- Keep the registry and configuration private to the local user where practical.
- No telemetry, network calls, background processes, or automatic transcript uploads.
- Do not log source code, credentials, meeting contents, or other sensitive project information.

The context may contain confidential client information. The design must not assume that external storage automatically makes its use permissible; data handling remains subject to the relevant client's policies.

## 7. Architecture principles

Keep responsibilities separate:

1. **Registry:** maps canonical repository identities to project IDs.
2. **Manifest:** describes a project's identity and external context location.
3. **Resolver:** determines which registered project, if any, matches the current repository.
4. **CLI:** handles user interaction and management commands.
5. **Claude adapter:** translates resolver output into Claude Code's supported session-start integration.
6. **Context templates:** provide a consistent structure for project knowledge.

The core registry and resolver should not depend on Claude Code. A future Codex adapter should be possible without rewriting the core.

Prefer deterministic lookup and filesystem search over semantic search or an LLM for the initial implementation.

## 8. Required planning output

Before recommending implementation, inspect the current environment and relevant repository/tooling documentation as appropriate. Do not assume the working directory is a suitable place for the new project; identify where the utility should live and ask me if the intended location is unclear.

Produce the following:

1. **Requirements summary:** distinguish explicit requirements from assumptions.
2. **Minimal architecture:** components, responsibilities, and data flow.
3. **Claude Code integration findings:** confirm the supported hook configuration, input format, and context-injection mechanism, with references to authoritative documentation where available.
4. **Storage and schemas:** proposed directory structure and example registry/manifest schemas.
5. **Repository matching strategy:** how to handle normal repositories, nested repositories, worktrees, moved paths, and ambiguous matches.
6. **Security review:** realistic risks and mitigations, especially context leakage between projects and unintended repository modifications.
7. **Implementation plan:** small, ordered milestones with specific files/modules, acceptance criteria, and tests for each milestone.
8. **Testing strategy:** unit tests, integration tests, and safe tests using temporary repositories and synthetic project data.
9. **Open questions:** ask only questions that materially affect the design.
10. **Explicit exclusions:** list features that should remain out of scope.

Challenge my proposed design where justified. Prefer existing supported mechanisms over custom infrastructure. Avoid speculative abstractions, excessive extensibility, and unnecessary dependencies.

Do not start implementation until I have reviewed and approved the plan.