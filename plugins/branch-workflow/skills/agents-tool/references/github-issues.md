# Tracking GitHub Issues in a project

Reference for the "GitHub Issues" section `agents-tool` scaffolds into `AGENTS.local.md`.
Not auto-loaded — read this when you're actually setting up or maintaining issue
tracking for a project, not on every `agents-tool` run.

## Why bother

- Agents need awareness of open issues before starting work
- Prevents working on already-reported bugs without context
- Helps prioritize tasks based on existing issues
- Cache format enables change detection without API spam

**Use `gh issue list` CLI commands, not GitHub MCP tools.** The CLI output format is
designed to be cached and compared for change detection — the JSON query format below
produces consistent, diff-friendly output that can be stashed in files.

**Where to document:**
- Private projects: in `AGENTS.local.md` or a private `AGENTS.md`
- Public projects: in `AGENTS.local.md` (gitignored), NOT the public `AGENTS.md`
- Cache location: `.agents/ISSUE_CACHE.md`, excluded via `.git/info/exclude` (`/.agents/`)

## Pattern 1: Inline

Best for projects with <10 open issues. Include directly in `AGENTS.local.md`:

```markdown
## GitHub Issues

**Last fetched:** 2025-12-15 23:30 UTC
**Recheck on or after:** 2025-12-22 (weekly)

**Important issues:**
- #42: Add dark mode support - assigned to me, in progress
- #38: Performance regression in v2.1 - needs investigation
- #27: Feature request: export to CSV - backlog

**All open issues (sorted by number):**
<!-- Fetch with: gh issue list --state open --json number,title,labels,updatedAt --limit 100 -->
#15 | Documentation outdated | labels: docs | updated: 2025-12-10
#27 | Feature: export to CSV | labels: enhancement | updated: 2025-12-14
#38 | Performance regression | labels: bug,p1 | updated: 2025-12-15
#42 | Add dark mode | labels: enhancement,in-progress | updated: 2025-12-15

**Directive to agents:**
When starting work on this project, compare the issue list above with latest from GitHub.
If last fetch is older than recheck date, or if you detect differences, update this section.
```

Refresh with:

```shell
{
  last_fetched=$(date -u +"%Y-%m-%d %H:%M UTC")
  recheck_after=$(date -u -d '+7 days' +"%Y-%m-%d")
  printf "**Last fetched:** %s
**Recheck on or after:** %s (weekly)

**All open issues (sorted by number):**
" "$last_fetched" "$recheck_after"
  gh issue list --state open --json number,title,labels,updatedAt \
    --jq 'sort_by(.number) | .[] | "#\(.number) | \(.title) | labels: \(.labels | map(.name) | join(",")) | updated: \(.updatedAt | split("T")[0])"'
} > /tmp/issues.txt
# Then manually merge into AGENTS.local.md
```

## Pattern 2: Separate cache file

Best for projects with >10 open issues. Keeps `AGENTS.local.md` clean.

In `AGENTS.local.md`:

```markdown
## GitHub Issues

**➡️ READ [.agents/ISSUE_CACHE.md](.agents/ISSUE_CACHE.md) to see all open issues categorized by label**

**Directive to agents:**
1. At session start: read the full ISSUE_CACHE.md file to see current state
2. Detect changes: compare issue numbers in cache vs live `gh issue list` output
3. Identify new issues: numbers in live output not in cache are NEW
4. Check staleness: if cache "Regenerate on or after" date has passed, regenerate
5. Regenerate: use the commands below, update `.agents/ISSUE_CACHE.md`
```

`.agents/ISSUE_CACHE.md`:

```markdown
# GitHub Issue Cache for <repo-name>

**Generated:** 2025-12-15 23:30 UTC
**Regenerate on or after:** 2025-12-22

## All Open Issues

#15 | Documentation outdated | labels: docs | updated: 2025-12-10
#27 | Feature: export to CSV | labels: enhancement | updated: 2025-12-14
#38 | Performance regression | labels: bug,p1 | updated: 2025-12-15
#42 | Add dark mode | labels: enhancement,in-progress | updated: 2025-12-15

## Important Issues

- #38: Performance regression - **HIGH PRIORITY**
- #42: Dark mode - in progress, targeting v2.2

## Recently Closed (last 30 days)

#40 | Fix login bug | closed: 2025-12-12
#35 | Update dependencies | closed: 2025-12-08
```

Add `/.agents/` to `.git/info/exclude` to keep the whole cache directory private.

Regeneration, filtered by label:

```shell
echo "## Bugs (needs attention)"
gh issue list --state open --label bug --json number,title,labels,updatedAt \
  --jq 'sort_by(.number) | .[] | "#\(.number) | \(.title) | labels: \(.labels | map(.name) | join(",")) | updated: \(.updatedAt | split("T")[0])"'

echo "## Non-Bug Issues"
gh issue list --state open --json number,title,labels,updatedAt \
  --jq 'sort_by(.number) | .[] | select(.labels | map(.name) | index("bug") | not) | "#\(.number) | \(.title) | labels: \(.labels | map(.name) | join(",")) | updated: \(.updatedAt | split("T")[0])"'
```

## Which pattern to use

- **Inline:** single file, self-contained, but clutters `AGENTS.local.md` past ~10 issues
- **Separate cache file:** keeps `AGENTS.local.md` clean, easy to regenerate, one more file to manage

## Detecting changes

```shell
gh issue list --state open --json number --jq '.[].number' | sort -n > /tmp/current_issues.txt
grep "^#[0-9]" .agents/ISSUE_CACHE.md | cut -d'|' -f1 | tr -d '# ' | sort -n > /tmp/cached_issues.txt
if ! diff -q /tmp/current_issues.txt /tmp/cached_issues.txt > /dev/null; then
  echo "Issues have changed - cache needs update"
  diff /tmp/cached_issues.txt /tmp/current_issues.txt
else
  echo "Issue cache is current"
fi
```

## Agent directive

When you encounter an issue cache in `AGENTS.local.md` or `.agents/ISSUE_CACHE.md`:

1. Check if the "recheck" date has passed
2. If yes, or if making changes to the project, fetch current issues
3. Compare with cached listing (issue numbers as fingerprint)
4. If differences found, notify the user and optionally update the cache
5. Highlight new issues or newly closed issues in output

Self-sustaining once set up — only needs manual intervention if the script fails or the
format changes.
