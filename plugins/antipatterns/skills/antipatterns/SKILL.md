---
name: antipatterns
description: >
  Glossary of David's shorthand nicknames for recurring antipatterns (SNAiTF, PPP, ...).
  Use when he references one of these terms, or when reviewing a commit/change
  description or a block of print/log/echo statements for these specific issues.
---

# antipatterns

Terse, memorable names for antipatterns David flags often, so he can say the shorthand
on any system and be understood without re-explaining.

## SNAiTF

"Sir, not appearing in this film." A commit/change description (or doc) mentioning
something that was never actually part of the change/repo — no reason a reader would
expect it there. E.g. a fresh scaffold's description saying "not yet migrated: X" when X
was never part of this repo to begin with.

Fix: cut the line. Describe the change relative to what actually preceded it, not
relative to unrelated context the reader doesn't have.

## PPP (print-print-print)

Also known as "log-log-log." Multiple consecutive `echo`/`printf`/`Logger.log` calls
that should be one atomic multi-line call instead — breaks vertical alignment, and
output can interleave with concurrent processes between calls.

Fix: build the full message (with real embedded newlines, not `\n` escapes) and emit it
in a single call.

## Adding new terms

Keep entries short: name, one-line definition, one-line fix. This file is meant to stay
skimmable, not become a full style guide — link to the fuller rule (e.g. in
`AGENTS.global.md`) instead of restating it here.
