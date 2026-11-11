# Multi-Agent Repo Setup (Claude Code + Antigravity + Beads)

One-shot prompt that turns a Claude Code-centric repo into a shared workspace where **Claude Code** and **Antigravity** read one rulebook (`AGENTS.md`) and hand off work through **Beads (`bd`)**. Not a skill: it runs once per repo and you always invoke it by name, so a plain file the agent reads is enough.

## Usage

In any agent, from the target repo (point it at wherever you keep this file - e.g. your clone of this repo):

```text
Read and execute <path-to-clone>/prompts/multi-agent-repo-setup/prompt.md
```

Add "use the conservative git profile" for repos that are not solo-maintained.

## Why This Layout

| Agent | Auto-loads on session start | Expands `@file` imports? |
| :--- | :--- | :--- |
| **Claude Code** | `CLAUDE.md` | Yes (`@AGENTS.md` inlines the target file) |
| **Antigravity** | `AGENTS.md`, `GEMINI.md`, `.agents/` rules | No |
| **Beads (`bd`)** | Manages `<!-- BEGIN/END BEADS INTEGRATION -->` in `AGENTS.md` | N/A |

Target state:

- `AGENTS.md` - regular file, under 12,000 bytes, the single source of truth: project rules, handoff rules, and the Beads block.
- `CLAUDE.md` - regular file containing only `@AGENTS.md`. Claude Code inlines `AGENTS.md`; no symlink, no duplication, no loop.

## Pitfalls (Found Setting Up Driftboard)

1. **`AGENTS.md -> CLAUDE.md` symlink trap.** A file-level symlink stores only the target name. Edits to `AGENTS.md` write through to `CLAUDE.md`, so `git add AGENTS.md` stages nothing (the link itself didn't change) while `CLAUDE.md` sits dirty. Atomic-save writers (`perl -pi`, temp-file-then-rename) also silently replace the link with a regular file, forking the two.
2. **Mutual imports.** Putting "see `CLAUDE.md`" in `AGENTS.md` costs Antigravity a tool call every session (it doesn't expand `@` imports) and risks circular inclusion in tools that do.
3. **12,000-character budget.** Antigravity caps or truncates rule files around 12,000 characters. Adding ~1,200 bytes of handoff and Beads sections to an 11,158-byte `CLAUDE.md` hit 12,351 until prose was tightened and em-dashes (3 bytes in UTF-8) became hyphens (1 byte). The prompt pre-authorizes that trim.
4. **Terminal paste corruption.** Pasting a multi-line prompt into a narrow pane garbled ~50-column chunks (`bd rememb..."`, `sessioor "don't push"`) and cost a round-trip. Hence a file the agent reads instead of a paste.

## Lessons Folded Into the Prompt

| Problem in first run | Fix in `prompt.md` |
| :--- | :--- |
| Assumed `AGENTS.md` was a regular file; it was a symlink | Step 0 runs `ls -la`; Step 1 handles symlink / missing / both-regular / already-correct |
| "Commit only `AGENTS.md`" blocked the `CLAUDE.md` conversion | Step 5 stages `CLAUDE.md` too when Step 1 changed it |
| "Keep under 12,000" with no plan for going over | Step 3 pre-authorizes em-dash swap and prose tightening |
| H1 stayed `# CLAUDE.md` | Step 2 sets `# AGENTS.md` |
| Step 4 would overwrite an existing `decision-*` memory, breaking the Decisions rule it just added | Step 4 checks `bd memories` first |
| Team-maintainer hardcoded for every repo | Profile is a default, overridable at invocation |
| Nested ```` ``` ```` fences broke the inserted block | Outer fence is `~~~~` |
