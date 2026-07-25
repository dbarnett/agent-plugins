# AGENTS.md — agent-plugins

CRITICAL: If `AGENTS.local.md` is present in this repo, you MUST read it: @AGENTS.local.md
(It's gitignored/untracked — machine/session-specific notes. Not present yet? Fine, ignore this line.)

See README.md for what this repo is and how it's installed. See each plugin's
`skills/*/SKILL.md` for what that plugin does.

## Structure requirements

- Root needs both `.claude-plugin/marketplace.json` and `.cursor-plugin/marketplace.json`,
  listing the same plugins.
- Each plugin dir needs its own `.claude-plugin/plugin.json` and `.cursor-plugin/plugin.json`.
- Bundled scripts are referenced via `${CLAUDE_PLUGIN_ROOT}` (Claude) / `$CURSOR_PLUGIN_ROOT`
  (Cursor) — never hardcoded paths.
- Keep the root README.md to a one-line-per-plugin index; detailed usage belongs in each
  plugin's `skills/*/SKILL.md`, not duplicated at the top level.

## Security — before any push

This repo is public. Anything migrated in from private sources (dotfiles, other repos) MUST be
audited line-by-line for hardcoded paths/hostnames/usernames/emails/IPs, secrets/tokens (even
dummy/expired ones), and comments revealing internal infra. Flag findings explicitly before
deciding to strip vs. keep — never silently scrub-and-proceed.
