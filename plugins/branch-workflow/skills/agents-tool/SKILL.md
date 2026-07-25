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

The script lives in `scripts/agents-tool` next to this SKILL.md.

```
# In Claude
"${CLAUDE_PLUGIN_ROOT}/skills/agents-tool/scripts/agents-tool"
# In Cursor
"${CURSOR_PLUGIN_ROOT}/skills/agents-tool/scripts/agents-tool"

.../scripts/agents-tool"          # project mode: check/setup AGENTS files
.../scripts/agents-tool" --task   # also scaffold/review WIP scope in the change description
```

Idempotent — safe to run repeatedly. Run project mode routinely; run `--task` when
starting a new branch/change or when unsure if the current change's description still
reflects what you're doing.

## What project mode does

- Creates `AGENTS.local.md` (gitignored, machine/session-specific notes) if missing.
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

## Notes

- Detects jj vs. plain git automatically; behavior differs slightly (jj can `describe`
  directly, git stages the skeleton into `COMMIT_EDITMSG` for the next `git commit`).
- Uses a lighter-weight WIP flow (just a `WIP:` prefix check, no Scope/Out-of-Scope/TODOs
  template) for repos that don't really have discrete feature branches. Dotfiles/chezmoi
  repos get this automatically (detected via `.chezmoiignore`, `.chezmoi.toml.tmpl`, or
  well-known dotfiles paths). Any other repo can opt in the same way by adding a
  `.agents-nobranchy` marker file at the repo root — presence is what matters, content is
  currently unread (reserved for future config, e.g. a reason string).
