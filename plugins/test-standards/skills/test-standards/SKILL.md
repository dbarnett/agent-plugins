---
name: test-standards
description: >
  Language-agnostic test-quality principles: avoid overmocking, avoid splitting one
  logical assertion into multiple expect calls (expect-expect-expect), one test = one
  thing, and scope-documenting comments per test file. Use when writing, reviewing, or
  debugging tests, or when a test file's purpose/scope relative to sibling test files
  isn't already documented.
---

# test-standards

Language-agnostic principles for test quality, drafted here once and made concrete
per-project (see "Making this concrete per project" below) since abstract guidance is
easy for an agent to skim past.

## Core principles

- **One test = one thing.** Each test verifies a single behavior. Use parameterized
  tests for variations of the same logic, not copy-pasted near-duplicates.
- **Functional tests spanning multiple components** get their own file/directory,
  separate from unit tests.
- **Document each test file's scope in a header comment/docstring.** State what this
  file covers and — critically — what distinguishes it from sibling test files (e.g.
  "unit tests for the parser's error paths; see integration_test.* for the full
  pipeline"). A reader (human or agent) landing in a test file cold should immediately
  know whether their change belongs here or in a neighboring file, without having to
  read every test in every sibling file to figure out the boundary.

## Avoid field-by-field assertions (expect-expect-expect)

**Prefer asserting one complete value over multiple expects on its pieces.**

Bad — splits one logical check into several assertions, illustrated in TypeScript:
```ts
expect(result.map(h => h.text)).toEqual(['A', 'B']);
expect(result.map(h => h.lineIndex)).toEqual([1, 3]);
// or
expect(result.ok).toBe(false);
expect(result.error).toMatch(/something/);
```

Good — assert the whole value at once:
```ts
expect(result).toEqual([
  { lineIndex: 1, level: 2, text: 'A' },
  { lineIndex: 3, level: 2, text: 'B' },
]);
expect(result).toMatchObject({ ok: false, error: expect.stringMatching(/something/) });
```

**Why:** field-by-field failures lose context — "expected 'foo' to be 'bar'" tells you
far less than seeing the whole mismatched object at once. Multiple `expect` calls
checking properties of the *same* value almost always collapse into one structural
assertion (`toEqual`/`toMatchObject` in JS, `assertEquals` on a full record/dataclass in
other languages, etc).

For deterministic-but-complex output, prefer inline/golden snapshots over hand-coding
the expected string.

## Avoid overmocking

**Test real behavior, not mocks.**

- Prefer real implementations over mocks whenever practical — real objects, real
  methods, real integrations. Only mock true external dependencies (network APIs,
  databases, filesystem, wall-clock time).
- Mocking internal collaborators masks real bugs and makes tests brittle to refactors
  that don't change actual behavior.
- Test public APIs and observable outcomes, not the exact sequence of internal calls.
  Verify state changes/side effects, not implementation details.
- Test-double taxonomy, use the right one: **stub** (canned responses), **fake**
  (working shortcut implementation, e.g. in-memory DB), **spy** (records calls for
  verification), **mock** (pre-programmed expectations + verifies them), **dummy**
  (passed but never used).

## Making this concrete per project

These principles are abstract by design — but abstract guidance is exactly what's easy
for an agent to nod at and then violate anyway, especially the "field-by-field" and
"overmocking" ones in an unfamiliar language/framework where the concrete shape of a
violation looks different than the TS examples above.

**The first time you're writing/reviewing tests in a project and these principles would
apply:** check whether that project's `AGENTS.md` (if the conventions should be shared
with other contributors) or `AGENTS.local.md` (if the project is private/solo) already
has a "Testing Examples" section with concrete good/bad examples in the project's actual
language/framework.

- **If yes:** use it as-is, don't re-derive.
- **If no, or stale for the frameworks actually in use:** write one now, before
  finishing the test-writing task. Translate each principle above into the project's
  real language/framework/mocking-library, using an actual (or realistic) example from
  that codebase, not a generic placeholder. This is a one-time cost per project that
  makes the guidance self-reinforcing for every future session in that project, instead
  of restating easy-to-skim prose every time.

Shared/team projects get this in the committed `AGENTS.md` so every contributor's agent
benefits, not just the one session that happened to write it.
