# Mneme

Portable project knowledge for AI coding sessions, kept outside the repository it describes. The main pain point is onboarding: someone picking up or joining a project shouldn't start from nothing. A smaller goal that's still worthwhile is moving context between the owner's own computers.

## Status

**Planning only. No code exists, and none should be written until the owner explicitly says to start implementation.** The original milestone plan was superseded when the scope grew, and planning has reopened.

## Read first

| File | What it holds |
|---|---|
| [docs/decisions.md](docs/decisions.md) | Decision log, open questions and working ideas. Start here |
| [docs/research-existing-tools.md](docs/research-existing-tools.md) | Comparison of existing tools and lessons for the open questions |
| [docs/implementation-plan.md](docs/implementation-plan.md) | Superseded milestone plan, kept for reference |
| [docs/design.md](docs/design.md) | Storage, schemas and repository matching from the first plan |
| [docs/claude-code-integration.md](docs/claude-code-integration.md) | Verified Claude Code hook behaviour, with sources |
| [docs/security.md](docs/security.md) | Risks and mitigations from the first plan |
| [project-brief.md](project-brief.md) | The original brief, kept as a record |

## Where we left off (2026-11-07)

1. The existing-tools research is done. The finding: nothing found does all four of nothing written to the client repo, no server, knowledge portable to another person, and verified knowledge kept separate from agent drafts.
2. **Next step, waiting on the owner's go-ahead:** answer three questions, reading docs before installing anything.
   - Can OKF Agent Memory bundles live outside the repository, and should Mneme adopt its format instead of inventing one?
   - Does ai-memory write anything into the repository, and can it run without a shared server?
   - How does Basic Memory handle folder location and sharing?
3. Then work through open questions 1 (what a piece of knowledge is) and 2 (what makes it trustworthy) together.

## Settled direction

- **Never write to a client repository.** Mneme links to a repository from outside it.
- **At session start, add only a short note** saying a knowledge base exists and where, in the style of Graphify. Project documents are never preloaded, so unrelated work in a project folder costs almost no context.
- **Claude should find knowledge implicitly.** In a fresh session, "tell me about payments" should work without the owner pointing at files.
- **Validate knowledge by type.** Check claims about code against cited files, in the style of Copilot memory. Decisions need a named person and date, and processes may need a review date.
- **Planned runtime is Node 26**, pinned in `.nvmrc`, using plain JavaScript with JSDoc types and no dependencies. Nothing is built yet.

## Known constraint

User-level `SessionStart` hooks don't run in cloud sessions, because cloud sessions don't read `~/.claude`. For now the owner starts sessions locally in the desktop app and continues them on their phone through Remote Control, which keeps the session running on their machine with hooks in effect. Starting sessions from the phone, which would be cloud sessions, may come later and remains an open question.

## Working with the owner

- Planning is deliberately thorough. Challenge assumptions, and flag risks and trade-offs rather than rushing to build.
- The owner makes commits. Don't commit or push unless asked.
- The repository may become public. Keep client names and other machine-specific details out of it, and use synthetic examples such as "Acme / Payments".
- Record decisions in `docs/decisions.md` as they're made, so the reasoning isn't lost.
