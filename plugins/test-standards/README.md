# test-standards

Language-agnostic test-quality principles plus a mechanism for drafting project-local,
language-concrete testing conventions. See
[`skills/test-standards/SKILL.md`](skills/test-standards/SKILL.md) for the actual skill
content.

## Why build this instead of using an existing skill

- [petrkindlmann/qa-skills](https://github.com/petrkindlmann/qa-skills) — 50 QA/test
  skills, including a solid `unit-testing` SKILL.md (test-doubles taxonomy,
  behavior-vs-implementation, Jest/Vitest/pytest specifics). Good reference-quality
  content, but no `.claude-plugin`/`.cursor-plugin` manifest at all — it's a flat
  `skills/` tree meant for their own separate `qaskills.sh` installer, not natively
  installable via either marketplace. Doesn't cover the specific expect-expect-expect
  field-splitting antipattern or per-test-file scope-documentation convention this
  plugin exists for.
- General web search for testing-quality skills otherwise turned up generic
  "Claude Code testing best practices" content, not a packaged skill.

Given the scope here is fairly specific (this repo owner's own established conventions
around assertion structure, mocking discipline, and test-file scope documentation) and
nothing found matched closely enough to adopt as-is, built from scratch, adapting from
`~/.agents/rules/testing.md` (pre-existing personal conventions) and adding the
per-project concrete-example mechanism and the test-file-scope-comment convention, which
weren't written down anywhere before this.

## Design note: language-agnostic vs. concrete

The core principles here are deliberately abstract (framework/language-agnostic), but
abstract guidance is easy for an agent to skim past. Rather than trying to enumerate
every language/framework's syntax in this skill (unbounded scope, guaranteed to be
incomplete), the skill instructs the agent to draft a project-local "Testing Examples"
section (in that project's `AGENTS.md` or `AGENTS.local.md`) with concrete examples in
the project's actual language, the first time the principles are relevant there. See
"Making this concrete per project" in the skill itself.

## Possible future enhancements

Not started, just flagged so they're not lost:

- **More antipatterns as they're caught.** This plugin exists because an agent was
  caught overmocking and doing expect-expect-expect in the same session; expect the
  principle list to grow opportunistically the same way.
- **Revisit petrkindlmann/qa-skills periodically** — if they ever add proper plugin
  manifests and the content stays high quality, might be worth depending on/adopting
  parts of it instead of maintaining our own.
