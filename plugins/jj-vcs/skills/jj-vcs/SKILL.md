---
name: jj-vcs
description: >
  Jujutsu (jj) version control: what it is, how it differs from git, everyday commands,
  selective squash/split, and conflict resolution. Use when working in a repo with a
  `.jj/` directory, when asked to perform version control operations there, or when
  asked "what is jj" / "how do I do X in jj vs git".
---

# jj-vcs

## What is jj?

Jujutsu ([jj-vcs.dev](https://jj-vcs.dev/)) is a version control system with a friendlier
model than git, while remaining git-compatible (most jj repos are "colocated" with a real
`.git` directory underneath — `jj git push`/`jj git fetch` talk to normal git remotes).

Key differences from git:

- **Continuous snapshots.** The working copy is *always* a change (`@`) that auto-updates
  as you edit files — no `git add`, no staging area.
- **Changes, not commits.** You work with "changes" (stable change IDs, e.g. `kkmpptxz`)
  rather than commits (which are the underlying, transient git-facing identity). Change
  IDs persist across rebases/edits; commit IDs don't.
- **Editing is implicit.** There's no `git commit --amend` — you just edit files, and
  they're part of `@` (or whichever change you `jj edit`'d) automatically.
- **Bookmarks, not branches.** Like git branches, but they don't auto-advance — you move
  them explicitly (`jj bookmark set`).
- **First-class conflicts.** A conflicting rebase doesn't stop and block you — the
  conflict is recorded *inside* the change, and you can keep working or resolve it later.

## Git → jj command mapping

| git | jj |
| --- | --- |
| `git status` | `jj status` |
| `git add <path>` | nothing — already tracked automatically |
| `git commit -m "..."` | `jj describe -m "..."` (describes `@`), or `jj new -m "..."` to start the *next* change |
| `git commit --amend` | just edit files — `@` updates automatically |
| `git log` | `jj log` |
| `git diff` | `jj diff` |
| `git checkout <branch>` | `jj new <bookmark>` (new change on top) or `jj edit <change-id>` (edit that change directly) |
| `git branch <name>` | `jj bookmark create <name> -r @` |
| `git push` | `jj git push` |
| `git pull` / `git fetch` | `jj git fetch` |
| `git stash` | usually not needed — `jj new` to set the in-progress work aside as its own change, come back with `jj edit` |
| `git merge <branch>` | `jj new <rev1> <rev2>` (multi-parent change) |
| `git rebase -i` | usually not needed — jj auto-rebases descendants on any edit. Explicit: `jj rebase -d <dest>` |
| `git cherry-pick <rev>` | `jj duplicate <rev>` (copy) or `jj rebase -r <rev> -d <dest>` (move) |
| `git reset --hard` | `jj abandon` (current change) or `jj restore` (specific paths) |
| `git reflog` | `jj op log` (operation log — records every jj operation, not just commits) |

## Essential commands

```shell
jj status              # what's changed in @
jj show --stat         # @ summary with file stats
jj log                 # compact change graph
jj log --stat          # history with file stats
jj diff                # diff of @
jj new [-m "..."]      # start a new change on top of @ (stops editing the old one)
jj describe [-m "..."] # set/update @'s description
jj edit <change-id>    # make a specific change the one you're editing
jj prev / jj next      # move to parent / child change
jj bookmark create <name> -r @
jj bookmark set <name> -r @
jj bookmark list
jj git push [--bookmark <name>]
jj git fetch
jj undo                # undo the last jj operation
jj op log               # see all operations (recovery point for anything)
```

## Selective squash and split

Both `jj squash` and `jj split` accept explicit paths — you don't need the interactive
diff editor if you know what you want to move:

```shell
# Move just these paths from @ into its parent
jj squash <path> [<path>...]

# Move paths between two arbitrary changes
jj squash --from <rev> --into <rev> <path>...

# Split @: move the given paths out into a NEW child change, with the rest staying on @
jj split <path>... [-m "message for the split-out change"]
```

`jj split` (no paths, or with `-i`) opens an interactive diff editor for hunk-level
selection — genuinely interactive, no way around that. But `jj split <path>` with
explicit paths is fully non-interactive and works well for "these files are actually a
separate change" splits (this is exactly how the `agent-plugins` repo's own history was
cleaned up mid-session: `jj split hk.pkl -m "..."` pulled one file out into its own
change without touching an editor).

## Conflict resolution

Conflicts in jj are first-class: a change can *be* conflicted and you can still build on
top of it, describe it, or set it aside — nothing blocks until you're ready to resolve.

```shell
jj status               # conflicted files are listed here
jj log                  # conflicted changes are flagged
jj resolve               # interactively resolve the next conflicted file (opens a merge tool)
jj resolve --tool <tool>  # use a specific merge tool
jj resolve --list         # list conflicted files without resolving
```

If no merge tool is configured, jj writes standard conflict markers into the file
(similar to git's `<<<<<<<`/`=======`/`>>>>>>>`, with extra context for jj's
multi-parent conflicts) — edit the file directly to resolve, same as resolving a git
merge conflict by hand. Once every conflicted file is resolved, `jj status` stops
listing conflicts and the change is clean.

Because conflicts live inside the change (not the working copy state), `jj new` on top
of a conflicted change, `jj edit`-ing away and back, or even pushing (if your policy
allows it) — none of that loses the conflict or forces immediate resolution. Resolve
whenever it's convenient.

## Recovery

```shell
jj op log            # every operation, ever
jj undo              # undo the most recent operation
jj op restore <id>   # jump back to any prior operational state
```

Nothing is really lost — `jj op restore` recovers from almost any mistake, including
abandoned changes or bad rebases.

## Notes

- This skill covers jj mechanics only. For this repo family's `WIP:`-prefixed change
  description convention and when to run `agents-tool --task`, see the `branch-workflow`
  plugin instead — not duplicated here.
- Further reading: [jj-vcs.dev](https://jj-vcs.dev/) ·
  [tutorial](https://jj-vcs.dev/latest/tutorial/) ·
  [git comparison](https://jj-vcs.dev/latest/git-comparison/)
