# JQL Search and Output Formatting

## JQL Queries

Use `-q` to pass raw JQL queries for advanced filtering:

```bash
jira issue list -q "<JQL>" --plain --no-truncate
```

### Common JQL patterns

```bash
# Issues assigned to you, updated recently
jira issue list -q "assignee = currentUser() AND updated >= -7d ORDER BY updated DESC" --plain --no-truncate

# Issues in a specific project by status
jira issue list -q "project = <PROJECT> AND status = 'In Progress' ORDER BY priority DESC" --plain

# Issues created in a date range
jira issue list -q "project = <PROJECT> AND created >= '2026-04-20' AND created <= '2026-07-20' ORDER BY created DESC" --plain --no-truncate

# Issues with specific labels
jira issue list -q "project = <PROJECT> AND labels IN ('backend', 'api') ORDER BY updated DESC" --plain

# Unresolved bugs by priority
jira issue list -q "project = <PROJECT> AND type = Bug AND resolution = Unresolved ORDER BY priority DESC" --plain

# Issues resolved in last 30 days
jira issue list -q "project = <PROJECT> AND resolved >= -30d ORDER BY resolved DESC" --plain --no-truncate

# Issues by component
jira issue list -q "project = <PROJECT> AND component = 'UI' AND status != Closed" --plain

# Text search in summary or description
jira issue list -q "project = <PROJECT> AND (summary ~ 'migration' OR description ~ 'migration')" --plain

# Issues updated by a specific user (reporter or assignee)
jira issue list -q "project = <PROJECT> AND (reporter = 'user@example.com' OR assignee = 'user@example.com') AND updated >= -90d" --plain --no-truncate

# Subtasks of a parent
jira issue list -q "parent = <ISSUE-KEY>" --plain

# Issues in an epic
jira issue list -q "'Epic Link' = <EPIC-KEY>" --plain
```

### JQL operators reference

| Operator | Example | Description |
|----------|---------|-------------|
| `=` | `status = "Open"` | Exact match |
| `!=` | `status != "Closed"` | Not equal |
| `IN` | `status IN ("Open", "In Progress")` | In set |
| `NOT IN` | `priority NOT IN ("Low", "Lowest")` | Not in set |
| `~` | `summary ~ "migration"` | Contains text |
| `!~` | `summary !~ "test"` | Does not contain |
| `>=`, `<=` | `created >= -30d` | Date comparison |
| `IS EMPTY` | `assignee IS EMPTY` | Unset field |
| `IS NOT EMPTY` | `labels IS NOT EMPTY` | Has value |
| `WAS` | `status WAS "In Progress"` | Historical value |
| `CHANGED` | `status CHANGED FROM "Open" TO "Done"` | Field changed |

### JQL functions

| Function | Description |
|----------|-------------|
| `currentUser()` | Authenticated user |
| `now()` | Current timestamp |
| `startOfDay()` | Start of today |
| `startOfWeek()` | Start of this week |
| `startOfMonth()` | Start of this month |
| `endOfDay()` | End of today |
| `membersOf("group")` | Members of a group |

## Date filter flags (alternative to JQL)

These flags are simpler than JQL for basic date filtering:

```bash
# Relative dates
jira issue list --created -10d --plain                  # created in last 10 days
jira issue list --updated -2w --plain                   # updated in last 2 weeks
jira issue list --created month --plain                 # created this month
jira issue list --updated today --plain                 # updated today

# Absolute dates
jira issue list --created-after 2026-04-20 --plain
jira issue list --created-before 2026-07-20 --plain
jira issue list --updated-after 2026-06-01 --updated-before 2026-07-01 --plain
```

## Output formats

### Plain table (default for scripts)

```bash
jira issue list --plain                                 # tab-separated
jira issue list --plain --no-headers                    # no header row
jira issue list --plain --no-truncate                   # full column width
jira issue list --plain --delimiter ","                  # custom delimiter
```

### JSON output

```bash
jira issue list --raw                                   # full JSON
jira issue view <KEY> --raw                             # single issue JSON
```

### CSV output

```bash
jira issue list --csv                                   # CSV format
```

### Column selection

```bash
jira issue list --columns KEY,SUMMARY,STATUS,PRIORITY,ASSIGNEE,UPDATED --plain --no-truncate
```

Available columns: `TYPE`, `KEY`, `SUMMARY`, `STATUS`, `ASSIGNEE`, `REPORTER`, `PRIORITY`, `RESOLUTION`, `CREATED`, `UPDATED`, `LABELS`
