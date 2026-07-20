---
name: jira-issues
description: Use the Jira CLI to manage Jira issues, sprints, epics, and boards. Use when the user wants to list issues, view tickets, check sprint status, search with JQL, or review Jira activity.
---

# jira - Jira CLI

`jira` is a CLI for managing Jira issues, sprints, epics, and projects from the terminal.

## Safety Rule

This skill is **read-only by default**. You may freely run read commands (`list`, `view`, `me`, `serverinfo`, `open`).

**Before performing any write operation** — creating, editing, transitioning, assigning, commenting, cloning, deleting, or logging work — you **MUST ask the user for explicit permission** and describe exactly what you are about to do. Never assume permission for write operations.

Write operations include: `create`, `edit`, `move`, `assign`, `comment add`, `clone`, `delete`, `link`, `unlink`, `watch`, `worklog add`, `sprint add`, `sprint close`, `epic create`, `epic add`, `epic remove`.

## Required CLI Tools

This skill requires `jira` (jira-cli by ankitpokhrel).

Check availability:
```bash
jira version
jira me
```

If not installed: `go install github.com/ankitpokhrel/jira-cli/cmd/jira@latest`, then `jira init`.

The `jira` binary may be at `~/go/bin/jira` — use the full path if it is not on PATH.

## Common Workflows

### Check your identity and server

```bash
jira me
jira serverinfo
```

### List and search issues

> Full issue command reference: [ref-issues.md](ref-issues.md)

```bash
jira issue list -a "$(jira me)" --plain --no-truncate
jira issue list -a "$(jira me)" -s "In Progress" --plain
jira issue list --created-after 2026-04-20 --plain --no-truncate
jira issue list -q "project = <PROJECT> AND assignee = currentUser() ORDER BY updated DESC" --plain --no-truncate
```

### View an issue

```bash
jira issue view <ISSUE-KEY> --plain
jira issue view <ISSUE-KEY> --comments 5 --plain
jira issue view <ISSUE-KEY> --raw
```

### Sprint and epic management

> Sprint and epic reference: [ref-sprints.md](ref-sprints.md)

```bash
jira sprint list --state active --plain
jira sprint list <SPRINT_ID> --plain --no-truncate
jira epic list --plain
jira epic list <EPIC-KEY> --plain --no-truncate
```

### Get child issues (subtasks) of a parent

```bash
jira issue list -q "parent = <ISSUE-KEY>" --plain --no-truncate
jira issue list -q "parent = <ISSUE-KEY> AND status != Done" --plain
jira issue list -P <PARENT-KEY> --plain --no-truncate
```

### Search with JQL

> JQL and output format reference: [ref-search.md](ref-search.md)

```bash
jira issue list -q "assignee = currentUser() AND updated >= -7d ORDER BY updated DESC" --plain --no-truncate
jira issue list -q "project = <PROJECT> AND status = 'In Progress'" --plain
```

### Open issue in browser

```bash
jira open <ISSUE-KEY>
```

### Output formats

- `--plain` — tab-separated table (default columns: TYPE, KEY, SUMMARY, STATUS, ASSIGNEE)
- `--plain --no-truncate` — show all columns without truncation
- `--columns KEY,SUMMARY,STATUS,PRIORITY,UPDATED` — select specific columns
- `--raw` — full JSON output
- `--csv` — CSV format

### Date filters

Relative: `today`, `week`, `month`, `year`, `-10d`, `-2w`, `-3h`
Absolute: `2026-04-20` or `2026/04/20`

```bash
jira issue list --created-after 2026-04-20 --updated-after 2026-06-01 --plain
jira issue list --created month --plain
jira issue list --updated -2w --plain
```

## Self-Learning Rule

When you encounter an unfamiliar `jira` command or need to verify flags, always run:

```bash
jira <command> --help
```
