Set up this repo so Claude Code and Antigravity share a single rulebook (`AGENTS.md`) and hand off work through beads (`bd`).

Git profile: `team-maintainer` (sole maintainer) unless the user said otherwise when invoking this prompt. If they asked for `conservative`, skip the "Git Profile" section in Step 2 and all of Step 4.

Execute these steps in order.

## Step 0 - Verify environment before changing anything

1. List every instruction or rules file you loaded for this workspace (`AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.agents/`, global rules) and quote the first line of each.
2. Run `ls -la AGENTS.md CLAUDE.md GEMINI.md` (for whichever exist) to check whether any is a symlink or missing.
3. Run `bd version` and `bd prime`, and check that `.beads/` exists. If `bd` is missing or this repo has no `.beads/` directory, stop and tell me. Do not continue.

## Step 1 - Make `AGENTS.md` the single rulebook and `CLAUDE.md` an `@AGENTS.md` import

Target state:

- `AGENTS.md` is a regular file (not a symlink) containing the full rulebook.
- `CLAUDE.md` is a regular file (not a symlink) containing only `@AGENTS.md` (with a trailing newline). Claude Code expands the import; Antigravity reads `AGENTS.md` directly.

How to get there from whatever current state exists:

- If `AGENTS.md` is a symlink to `CLAUDE.md`: `rm -f AGENTS.md`, `cp -f CLAUDE.md AGENTS.md`, then overwrite `CLAUDE.md` with `@AGENTS.md`.
- If `AGENTS.md` does not exist and `CLAUDE.md` has the rules: `cp -f CLAUDE.md AGENTS.md`, then overwrite `CLAUDE.md` with `@AGENTS.md`.
- If both exist as regular files with different rules: merge `CLAUDE.md`'s rules into `AGENTS.md` (no duplicates), then overwrite `CLAUDE.md` with `@AGENTS.md`.
- If `AGENTS.md` already has the full rules and `CLAUDE.md` already contains only `@AGENTS.md`, leave `CLAUDE.md` untouched.
- If `GEMINI.md` exists, do not edit it. Report its contents in Step 6 so I can decide whether to fold it into `AGENTS.md`.

## Step 2 - Update the top of `AGENTS.md`

1. Set the top-level heading of `AGENTS.md` to `# AGENTS.md`.
2. Directly under `# AGENTS.md`, replace any Claude-only intro line with:

   `Shared rulebook for every coding agent in this repo (Claude Code, Antigravity). Edit only this file - CLAUDE.md just imports it. Keep it under 12,000 characters (Antigravity's rule-file cap).`

3. Directly under that line (and ABOVE any `<!-- BEGIN BEADS INTEGRATION` marker), add the sections between the `~~~~` lines below. Never edit text between `<!-- BEGIN BEADS INTEGRATION` and `<!-- END BEADS INTEGRATION -->`. If a section with the same purpose already exists, merge into it instead of duplicating. Use plain hyphens (`-`), not em-dashes. Do not hard-wrap paragraphs.

~~~~markdown
## Start of Session

1. Run `bd prime` before touching anything. Claude Code does this via hook; Antigravity must run it itself.
2. `bd ready`, pick a bead, `bd update <id> --claim`.

## Before You Stop (Handoff)

Always, even mid-task, leave a note on the bead you worked:

```bash
bd update <id> --append-notes "DONE: ... / NOT DONE: ... / NEXT: ..."
```

Use `--append-notes`, never `--notes` (it silently replaces). Questions for the maintainer go in the same note, prefixed `QUESTION:`.

## Decisions

Settled choices are stored as `bd remember --key decision-<slug> "..."` and show up in `bd prime`. Never change or `bd forget` a `decision-*` memory. If you disagree, say so in your handoff note and keep following it.

## Concurrency

One agent per working tree at a time. `--claim` locks a task, not files. Parallel work needs a separate git worktree.

## Git Profile: Team-Maintainer

This repo opts into the `team-maintainer` profile (sole maintainer). Agents may close beads, commit, and push (`git push`, `bd dolt push`) at session close. A "don't commit" or "don't push" instruction in the current session still wins. Never force-push.
~~~~

## Step 3 - Enforce the 12,000-character limit

Run `wc -c AGENTS.md` and report the byte count. If it is over 12000:

- Replace UTF-8 em-dashes (3 bytes each) with plain hyphens (`-`) outside the `BEGIN/END BEADS INTEGRATION` block.
- Tighten verbose phrasing in existing non-Beads sections, without removing any rules or commands, until `wc -c AGENTS.md` is under 12000.

## Step 4 - Record the git profile decision in Beads

Run `bd memories decision-git-profile`. If a `decision-git-profile` memory already exists, do not change it (per the Decisions rule) - report it instead. Otherwise run:

```bash
bd remember --key decision-git-profile "Repo uses team-maintainer git profile (sole maintainer, decided <YYYY-MM-DD>): agents may close beads, commit, git push, bd dolt push at session close. No force-push."
```

## Step 5 - Commit (do not push)

Stage only `AGENTS.md` (and `CLAUDE.md` if it was modified in Step 1). Do not stage any other dirty files. Commit with the message:

```text
Add shared agent handoff rules and git profile
```

Do not push.

## Step 6 - Report and hand off

1. Show the output of:
   - `git show --stat HEAD`
   - `ls -la AGENTS.md CLAUDE.md`
   - `wc -c AGENTS.md`
   - `grep '^#' AGENTS.md`
2. If `GEMINI.md` exists, show its contents.
3. Follow the handoff rule: check `bd list --status=in_progress`. If you worked under a bead, append a `DONE: ... / NOT DONE: ... / NEXT: ...` note to it. If there was no bead, tell me that instead.
