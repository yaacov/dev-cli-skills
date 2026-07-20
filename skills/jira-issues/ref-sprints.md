# Sprint and Epic Commands

## Sprints

### List sprints

```bash
jira sprint list --plain                                                    # top 50 sprints
jira sprint list --state active --plain                                     # active sprints only
jira sprint list --state closed --plain                                     # closed sprints
jira sprint list --state future --plain                                     # future sprints
jira sprint list --current --plain                                          # current sprint's issues
jira sprint list --prev --plain                                             # previous sprint's issues
jira sprint list --next --plain                                             # next sprint's issues
```

### View sprint issues

```bash
jira sprint list <SPRINT_ID> --plain --no-truncate                          # issues in sprint
jira sprint list <SPRINT_ID> --columns KEY,SUMMARY,STATUS,ASSIGNEE --plain
jira sprint list <SPRINT_ID> -q "assignee = currentUser()" --plain          # your issues in sprint
jira sprint list <SPRINT_ID> -C "backend" --plain                           # by component
jira sprint list <SPRINT_ID> --order-by status --plain                      # ordered by status
```

### Sprint column selection

For sprint listing: `ID`, `NAME`, `START`, `END`, `COMPLETE`, `STATE`
For sprint issues: same as issue columns (`KEY`, `SUMMARY`, `STATUS`, `ASSIGNEE`, etc.)

```bash
jira sprint list --columns ID,NAME,STATE,START,END --plain
```

### Write operations (require user permission)

```bash
jira sprint add <SPRINT_ID> <ISSUE-1> <ISSUE-2>                            # add issues to sprint (max 50)
jira sprint close <SPRINT_ID>                                               # close a sprint
```

## Epics

### List epics

```bash
jira epic list --plain                                                      # top 100 epics
jira epic list --columns KEY,SUMMARY,STATUS --plain
jira epic list -s "In Progress" --plain                                     # by status
```

### View epic issues

```bash
jira epic list <EPIC-KEY> --plain --no-truncate                             # issues in epic
jira epic list <EPIC-KEY> -s "In Progress" --plain                          # filtered by status
jira epic list <EPIC-KEY> -a "$(jira me)" --plain                           # your issues in epic
jira epic list <EPIC-KEY> --columns KEY,SUMMARY,STATUS,ASSIGNEE --plain
```

### Write operations (require user permission)

```bash
jira epic create -n "Epic name" -s "Epic summary" -b "Description" --no-input
jira epic create -n "Epic name" -s "Summary" -a "$(jira me)" -y Normal --no-input
jira epic add <EPIC-KEY> <ISSUE-1> <ISSUE-2>                                # add issues to epic (max 50)
jira epic remove <ISSUE-1> <ISSUE-2>                                        # remove issues from epic (max 50)
```

## Boards

### List boards

```bash
jira board list --plain
jira board list -p <PROJECT> --plain
```

## Projects

### List projects

```bash
jira project list --plain
```

## Releases

### List releases

```bash
jira release list --plain
```
