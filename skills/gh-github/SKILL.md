---
name: gh-github
description: Use the GitHub CLI (gh) to interact with GitHub repositories, pull requests, issues, code search, and the GitHub API. Use when the user wants to list PRs, view issues, check CI status, search code, or review GitHub activity.
---

# gh - GitHub CLI

`gh` is the official GitHub CLI for managing repositories, pull requests, issues, and more.

## Safety Rule

This skill is **read-only by default**. You may freely run read commands (`list`, `view`, `status`, `diff`, `checks`, `search`, `api GET`).

**Before performing any write operation** — creating, editing, commenting, merging, closing, reopening, reviewing, or deleting — you **MUST ask the user for explicit permission** and describe exactly what you are about to do. Never assume permission for write operations.

Write operations include: `create`, `edit`, `comment`, `merge`, `close`, `reopen`, `review`, `delete`, `lock`, `unlock`, `pin`, `unpin`, `transfer`, `api POST/PUT/PATCH/DELETE`.

## Required CLI Tools

This skill requires `gh` (GitHub CLI).

Check availability:
```bash
gh --version
gh auth status
```

If not installed: `brew install gh`, then `gh auth login`.

## Common Workflows

### Check your PR and issue status

```bash
gh pr status
gh issue status
gh status
```

### List and view pull requests

> Full PR command reference: [ref-pr.md](ref-pr.md)

```bash
gh pr list
gh pr list -s all -A <username> -L 20
gh pr view <number>
gh pr view <number> --json title,state,reviews,statusCheckRollup
gh pr diff <number>
gh pr checks <number>
```

### List and view issues

> Full issue command reference: [ref-issues.md](ref-issues.md)

```bash
gh issue list
gh issue list -s all -a <username> -L 20
gh issue view <number>
gh issue view <number> --json title,state,body,comments
```

### Search across GitHub

> Search and API reference: [ref-search.md](ref-search.md)

```bash
gh search prs --author=<username> --state=open
gh search issues --repo=<owner>/<repo> "search terms"
gh search code "functionName" --repo=<owner>/<repo>
```

### Target a specific repository

All repo-scoped commands support `-R OWNER/REPO`:

```bash
gh pr list -R <owner>/<repo>
gh issue list -R <owner>/<repo> -s open
```

### Read file content from a remote repository

```bash
gh api repos/<owner>/<repo>/contents/<path> --jq '.content' | base64 -d
gh api repos/<owner>/<repo>/contents/<path>?ref=<branch> --jq '.content' | base64 -d
gh api repos/<owner>/<repo>/contents/<directory> --jq '.[].name'
```

### Structured JSON output

Use `--json` with `--jq` for machine-readable output:

```bash
gh pr list --json number,title,state,author,updatedAt --jq '.[] | "\(.number) \(.title) (\(.state))"'
gh issue list --json number,title,labels --jq '.[] | select(.labels | length > 0)'
```

## Self-Learning Rule

When you encounter an unfamiliar `gh` command or need to verify flags, always run:

```bash
gh <command> --help
```
