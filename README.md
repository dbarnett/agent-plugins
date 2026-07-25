# agent-plugins

dbarnett's personal Claude Code / Cursor plugin marketplace.

## Plugins

- **[branch-workflow](plugins/branch-workflow/)** — want `agents-tool` (AGENTS.md setup +
  jj/git WIP scope-tracking) available as a plugin instead of a dotfiles script? Install this.
- **[antipatterns](plugins/antipatterns/)** — want an agent to recognize David's shorthand
  antipattern nicknames (SNAiTF, PPP, ...) without re-explaining them? Install this.

## Install

```
claude plugin marketplace add dbarnett/agent-plugins
claude plugin install <plugin-name>@dbarnett-agent-plugins
```

Cursor: same repo, reads `.cursor-plugin/marketplace.json` instead. See per-plugin READMEs/SKILL.md for details.
