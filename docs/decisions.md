# Decision Log

Decisions and open questions for Mneme, newest decisions first. Planning is open and implementation has not started.

## Goal

Make project knowledge portable between machines, clones and people, so that picking up or joining a project doesn't mean starting from nothing. The main pain point is onboarding and ramp-up, especially on projects with little documentation. Even if nobody else uses it, moving context between one person's own computers is a worthwhile result.

## Decisions

| Date | Decision | Why |
|---|---|---|
| 2026-11-03 | Research existing tools before further design | Avoid rebuilding something that exists, and learn from what others have tried |
| 2026-11-02 | Stay in planning; no implementation until the open questions are worked through | The project should not go down the wrong path early |
| 2026-11-02 | "Export" is an example of the requirement, not the requirement itself | The real need is portability; a file may not be the right mechanism |
| 2026-11-02 | Session start adds only a short note that a knowledge base exists and where | Opening Claude in a project folder for unrelated work shouldn't spend context; project documents are not injected |
| 2026-11-02 | Develop on Node 26, pinned in `.nvmrc` | Chosen by the project owner |
| 2026-11-02 | Node rather than PowerShell | Other people can use it if the project goes public |
| 2026-11-02 | `mneme` as the name for the command and folders | One name throughout |
| 2026-11-02 | Never write to the client repository | Client repositories may forbid AI configuration and extra files |

## Open questions

Ordered by how expensive each would be to get wrong.

1. **What is a piece of knowledge?** Free-form documents, or small entries with a source, date and status?
2. **What makes knowledge trustworthy?** Who or what can vouch for it, and how does it expire?
3. **How does knowledge move, and how do changes merge?** What's shared, what's personal, and what happens when two people disagree?
4. **How does knowledge get captured?** What triggers it, and who reviews it?
5. **How does Claude find it, and how do we know that works?** This needs a repeatable test set.
6. **Where are the confidentiality boundaries?** What may leave a machine or a client, and who decides?
7. **Who is it for, and how does it fit alongside what's already there?** Solo or team, Claude-only or tool-neutral, and how it relates to wikis, repository docs and auto memory.

## Working ideas

Not decided, recorded so they aren't lost. See also [Research: Existing Tools](research-existing-tools.md).

- **Validate knowledge by type.** Claims about code can record their source files and commit, then be flagged when those change. Decisions need a named person and a date. Processes may need a review date.
- **Review before knowledge is trusted.** Sessions draft notes into an inbox, and a person approves them.
- **Implicit search needs reads without prompts.** A scoped read hook could allow the current project's folder and block others.
- **Topic index.** An `index.md` that maps topics to files could help Claude find things without reading everything.
