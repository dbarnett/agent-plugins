---
name: agents-tool
description: >
  Idempotent setup/check tool for agent-friendly projects. Creates and refreshes
  AGENTS.md / AGENTS.local.md / CLAUDE.md, and (with --task) scaffolds WIP: scope/TODO
  tracking directly into the current jj change description (or .git/COMMIT_EDITMSG for
  plain git). Use when starting work on a new branch/change, when checking whether
  project agent-files are set up, or when the current change lacks a clear WIP
  description of scope and TODOs.
---

# agents-tool

## Purpose

Bundled script that keeps a project's agent-facing files (`AGENTS.md`, `AGENTS.local.md`,
`CLAUDE.md`) in sync, and — in task mode — keeps the *current* jj change (or pending git
commit) carrying a live `WIP:`-prefixed description of scope, out-of-scope, and TODOs.
Scope/TODO tracking lives in the change/commit description itself, not in a separate
tracked file — there is no `THIS_BRANCH.md`-style file to create or clean up.

## Running it

```
<skill-dir>/scripts/agents-tool          # project mode: check/setup AGENTS files
<skill-dir>/scripts/agents-tool --task   # also scaffold/review WIP scope
```

`<skill-dir>` is this SKILL.md's own directory:

- **Claude:** `"${CLAUDE_PLUGIN_ROOT}/skills/agents-tool"`
- **Cursor:** no equivalent root env var — use the directory this SKILL.md file is in
  (its path is in your context).

Idempotent — safe to run repeatedly. Run project mode routinely; run `--task` when
starting a new branch/change or when unsure if the current change's description still
reflects what you're doing.

## What project mode does

- Creates `AGENTS.local.md` (gitignored, machine/session-specific notes) if missing.
  Includes a scaffolded "GitHub Issues" section — see
  `references/github-issues.md` for the actual tracking/caching commands when you're
  setting that up (not auto-loaded, read it when relevant).
- Symlinks `CLAUDE.md` → `AGENTS.md`, and `AGENTS.md` → `AGENTS.local.md` by default
  (private pattern) unless a shared `AGENTS.md` already exists.
- Adds the right entries to `.git/info/exclude` (or the jj-colocated equivalent).
- Warns if `~/AGENTS.global.md` (a personal global template, not part of this plugin)
  has been updated more recently than the project's `AGENTS.local.md`.

## What `--task` mode does

- If the current jj change (or git `COMMIT_EDITMSG`) has no description, writes a
  `WIP: TODO - ...` skeleton with `## Scope`, `## Out of Scope`, `## TODOs`, `## Notes`
  sections directly into the description/commit-message.
- If a `WIP:`-prefixed description already exists, prompts to review whether it's still
  current.
- If the description has verbose/section content (`##`, `FIXME`, `TODO`, etc.) but is
  missing the `WIP:` prefix, warns loudly — that's not push-ready.
- Runs lightweight inline checks (e.g. scans changed files for `FIXME`) when a `WIP:`
  change is detected.
- Before exiting WIP, the expected workflow is: move useful notes out to code
  comments/docs/issues, then collapse the description to 2-5 lines and drop the `WIP:`
  prefix.
- Scaffolded WIP templates include an `agent:<tool>:<id>` line (optionally followed by a
  quoted name, e.g. `agent:cc:07db5c7d-18f4-438e-a22c-053f1de68d89 "some session name"`),
  auto-filled when the running agent's session ID can be inferred (currently: Claude Code
  CLI, via `CLAUDE_CODE_SESSION_ID`; override/supply manually via
  `AGENTS_TOOL_AGENT_ID=tool:id` and optionally `AGENTS_TOOL_AGENT_NAME=name`). ID and
  name are separate fields — ID is what gets matched for ownership, name is just for
  humans to eyeball. If a tool only exposes a name and no stable ID, use `?` as the ID
  placeholder: `agent:oc:? "some name"`.
  When reviewing an *existing* WIP change/commit, `--task` checks its `agent:` line(s)
  (matching on the ID field only) against this session's ID and warns loudly if this
  session isn't among them.
  **AGENTS: if that warning fires, do not edit the change further without the user's
  explicit permission** — it means a different agent session (possibly still running) is
  the one working on it. Branch off (`jj new`, or the git equivalent) instead, or confirm
  with the user first.
  Multiple `agent:` lines are valid when several sessions are deliberately collaborating
  on one change — add your own line alongside existing ones rather than replacing them.
  If no ID can be inferred and none is supplied (`AGENTS_TOOL_AGENT_ID`), `--task` refuses
  to scaffold a new WIP description at all — it prints the required env vars and stops,
  rather than writing a placeholder that could ship unfilled.
- **Two distinct ways to mutate another agent's change — `--task` only catches the
  first, and only *after* it's already happened:**
  1. **Explicit commands that target a change by revision**: `jj squash --into <rev>`,
     `jj edit <rev>`, `git commit --amend`, `git rebase --onto`/`-i` touching someone
     else's commit. Before running any of these against a specific revision, check *that
     revision's* description for an `agent:` line yourself (`jj log -r <rev> -T
     description`, `git log -1 <ref>`) — don't rely on `--task` to notice.
  2. **Working-copy file edits after switching onto an existing change**: in jj, the
     working copy *is* `@` — once `jj edit <rev>` (not `jj new`) has moved `@` onto an
     existing change, any later Edit/Write tool call silently mutates that change's
     content directly, no separate jj command needed. `--task` checks `@`'s `agent:` line,
     but only once run, and only after `@` already moved there. The actual guard is
     upstream of that: check a change's `agent:` line *before* `jj edit`ing onto it, and
     default to `jj new` (creates a fresh child, never mutates existing content) rather
     than `jj edit` unless deliberately resuming your own prior WIP change.

## Notes

- Detects jj vs. plain git automatically; behavior differs slightly (jj can `describe`
  directly, git stages the skeleton into `COMMIT_EDITMSG` for the next `git commit`).
- Uses a lighter-weight WIP flow (just a `WIP:` prefix check, no Scope/Out-of-Scope/TODOs
  template) for repos that don't really have discrete feature branches. Dotfiles/chezmoi
  repos get this automatically (detected via `.chezmoiignore`, `.chezmoi.toml.tmpl`, or
  well-known dotfiles paths). Any other repo can opt in the same way by adding a
  `.agents-nobranchy` marker file at the repo root — presence is what matters, content is
  currently unread (reserved for future config, e.g. a reason string).
