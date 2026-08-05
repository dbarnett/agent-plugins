---
description: Refresh AGENTS.local.md and .agents/ISSUE_CACHE.md if due for a refresh
argument-hint: [optional instructions, e.g. "force" or "GH issues only"]
---

<!-- Claude Code slash-command conventions ($ARGUMENTS, argument-hint frontmatter) used
here; not verified against Cursor's plugin spec for command compatibility. -->

Freshen this project's agent-facing local resources: `AGENTS.local.md` and, if set up,
`.agents/ISSUE_CACHE.md` (GitHub issue cache).

Instructions from the user for this run (may be empty — default to normal due-date-respecting
refresh):
$ARGUMENTS

Steps:

1. Read `AGENTS.local.md`'s `**Refresh By:**` date (top of file). If today is before that
   date, and nothing in the instructions above says to force/ignore dates, report "not due
   yet (Refresh By: <date>)" and stop — don't touch anything.
2. Unless instructions above say to focus only on GitHub issues: run the `agents-tool`
   skill's bundled script (`<skill-dir>/scripts/agents-tool`, where `<skill-dir>` is
   `${CLAUDE_PLUGIN_ROOT}/skills/agents-tool` on Claude, or that SKILL.md's own directory
   on Cursor) to refresh `AGENTS.md`/`CLAUDE.md`/symlinks/excludes. Then review
   `AGENTS.local.md`'s other sections (Current Task Context, Codebase Knowledge, etc.) for
   staleness and update what's actually gone stale — don't rewrite sections that are still
   accurate.
3. If `AGENTS.local.md` has a "GitHub Issues" section referencing `.agents/ISSUE_CACHE.md`
   (or an inline issue list), and it's due for a recheck (or the instructions above ask for
   GitHub issues specifically), regenerate it per the `agents-tool` skill's
   `references/github-issues.md` — read that file for the actual `gh issue list` commands
   and cache format, don't guess the format.
4. Update `AGENTS.local.md`'s `**Last Updated:**` and `**Refresh By:**` dates (and any
   per-section "Recheck on or after" dates you touched) to reflect this refresh.

Report a short summary of what was refreshed vs. left alone vs. skipped as not-due.
