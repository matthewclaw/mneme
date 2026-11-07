# Research: Existing Tools

Researched on 2026-11-03. Feeds the open questions in [decisions.md](decisions.md).

**How far to trust this:** it is based on project READMEs, docs and search summaries. No tool was installed or tested, and most claims, including benchmarks and star counts, are self-reported at the time of reading. "Not found" means not found in this research, not proof of absence.

## Summary

Git-backed, Markdown-based agent memory is a crowded space, but nothing found combines all four things Mneme needs:

1. Nothing written to the client repository.
2. No server to run.
3. Knowledge that can move to another person.
4. Trust levels that separate verified knowledge from agent drafts.

Several tools cover some of these, so the choice is not automatically to build from scratch.

## Tools compared

| Tool | Stored | Sharing | Trust and freshness | Fit |
|---|---|---|---|---|
| [ai-memory](https://github.com/akitaonrails/ai-memory) | Git-backed wiki on a self-hosted server. A `.ai-memory.toml` routing file is mentioned, and its location wasn't clear | Machines and teammates share one server, with attribution and audit log | Decay schedules, contradiction flags, sources shown. No default human review described | Strongest overlap, but needs a server and Rust binary. About 15k stars |
| [OKF Agent Memory](https://github.com/okf-memory/okf-agent-memory) | `./knowledge/` in the repo; user scope in `~/.okf/` | Encrypted hub sync, Git, registry | `generated` vs `verified`, `sources`, `status`, `stale_after`, `code_refs` | Best knowledge model found. In-repo by default. About 850 stars |
| [memobank](https://github.com/clawde-agent/memobank-cli) | Personal and workspace tiers outside the repo; project tier in it | Separate Git remote for workspace tier | Status driven by recall frequency; secret scan blocks publishing | Closest in shape, and also Node. Very early, 2 stars |
| [GitHub Copilot memory](https://github.blog/ai-and-ml/github-copilot/building-an-agentic-memory-system-for-github-copilot/) | GitHub-hosted, per repository | Repository access controls | Code citations checked just before use, then corrected if wrong | Proprietary, but the best validation idea found |
| [Basic Memory](https://docs.basicmemory.com/integrations/claude-code/) | Local or hosted cloud | Team sharing not mentioned on the page read | Not covered on the page read | Mature knowledge tool via MCP; needs a closer look |
| [claude-mem](https://www.datacamp.com/tutorial/claude-mem-guide) | Local SQLite | Not built in | Not covered in what was read | Automatic capture, personal only |
| [team-brain](https://github.com/Manavarya09/team-brain), [mex](https://github.com/mex-memory/mex), [repo-memory](https://pypi.org/project/repo-memory-mcp/), [project-context-memory](https://github.com/gorttham/project-context-memory) | In the repo | Git | Varies | Ruled out for client repositories, though the team-brain onboarding guide command is a relevant idea |

## Claude Code itself

- Auto memory is machine-local, and its folder is derived from the project's absolute path.
- [Issue #25739](https://github.com/anthropics/claude-code/issues/25739) asks for portable project memory, including export and import. It was open with recent activity and no official response when read, so the need is real and unmet upstream.
- [Cloud sessions](https://code.claude.com/docs/en/cloud-environments), including ones started from the mobile app, start from a fresh clone and don't read `~/.claude`, so user-level `SessionStart` hooks don't run there. A hook-based Mneme would only work in local sessions.
- The `autoMemoryDirectory` setting can redirect auto memory, and that issue collects workarounds and small transfer tools. These may already cover one person moving memory between machines, but not curated team knowledge. A built-in feature could still ship and overlap with Mneme.

## The compile pattern

[Karpathy's LLM Wiki](https://blog.starmorph.com/blog/karpathy-llm-wiki-knowledge-base-guide) matches the "compile context as you go" idea: immutable raw sources, an LLM-maintained Markdown wiki, a schema file, and ingest, query and lint operations. A [vendor variant](https://open-knowledge-docs.preview.inkeep.com/docs/workflows/karpathy-llm-wiki) adds separate provisional and canonical layers. These descriptions are secondary sources, not the original gist.

## What this suggests for the open questions

| Question | Lesson |
|---|---|
| 1. Unit of knowledge | Markdown files with structured metadata are the common pattern. OKF's fields are a strong starting point |
| 2. Trust | Combine two checks. Copilot-style citation checks suit claims about code, and OKF-style human sign-off suits what code can't prove. Be wary of recall frequency: it shows that knowledge is used, not that it's true |
| 3. Portability | Git is the common transport. A separate private remote fits client constraints, and a one-off export can be a thin layer on top |
| 4. Capture | Tools range from fully automatic to explicit commands. A pending queue or provisional layer is the recurring way to keep drafts apart from trusted knowledge |
| 5. Retrieval | A small index or brief plus keyword search is common and needs no vector database. Treat vector search as optional |
| 6. Confidentiality | Scan for secrets before anything is shared, and block rather than strip silently. Personal and shared tiers are a common boundary |
| 7. Fit | Mneme's clearest niche is client work where the repository can't change and a server isn't an option |

## To check next

- Can OKF bundles live outside the repository, and is its format worth adopting rather than inventing one?
- Does ai-memory write anything into the repository, and can it run without a shared server, for example one local instance with a Git remote?
- How does Basic Memory handle its folder location and sharing?
