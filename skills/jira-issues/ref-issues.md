# Issue Commands

## List issues

```bash
jira issue list --plain                                                     # all open issues in default project
jira issue list -a "$(jira me)" --plain --no-truncate                       # assigned to you
jira issue list -r "$(jira me)" --plain                                     # reported by you
jira issue list -s "In Progress" -s "New" --plain                           # by status (multiple)
jira issue list -y Critical --plain                                         # by priority
jira issue list -t Bug --plain                                              # by type
jira issue list -l "backend" --plain                                        # by label
jira issue list -C "UI" --plain                                             # by component
jira issue list -P <PARENT-KEY> --plain                                     # by parent
jira issue list -w --plain                                                  # issues you are watching
jira issue list --history --plain                                           # recently accessed
jira issue list -p <PROJECT> --plain                                        # specific project
```

### Column selection

Available columns: `TYPE`, `KEY`, `SUMMARY`, `STATUS`, `ASSIGNEE`, `REPORTER`, `PRIORITY`, `RESOLUTION`, `CREATED`, `UPDATED`, `LABELS`

```bash
jira issue list --columns KEY,SUMMARY,STATUS,PRIORITY,UPDATED --plain --no-truncate
```

### Pagination

```bash
jira issue list --paginate 0:100 --plain                                    # first 100 (default)
jira issue list --paginate 100:100 --plain                                  # next 100
```

### Output ordering

```bash
jira issue list --order-by updated --plain                                  # order by updated (default: created)
jira issue list --order-by priority --reverse --plain                       # ascending order
```

## View issue details

```bash
jira issue view <ISSUE-KEY> --plain                                         # plain text view
jira issue view <ISSUE-KEY> --comments 5 --plain                            # with last 5 comments
jira issue view <ISSUE-KEY> --raw                                           # full JSON
```

## Open in browser

```bash
jira open <ISSUE-KEY>
jira open <ISSUE-KEY> --no-browser                                          # just print the URL
```

## Write Operations (require user permission)

### Create an issue

```bash
jira issue create -t Task -s "Issue summary" -b "Description body" -y Normal --no-input
jira issue create -t Bug -s "Bug title" -a "$(jira me)" -l "backend" -y Critical --no-input
jira issue create -t Story -s "Story title" -P <PARENT-KEY> --no-input
echo "long description" | jira issue create -t Task -s "Title" --no-input
```

### Edit an issue

```bash
jira issue edit <ISSUE-KEY> -s "Updated summary" --no-input
jira issue edit <ISSUE-KEY> -b "Updated description" --no-input
jira issue edit <ISSUE-KEY> -y Critical --no-input
jira issue edit <ISSUE-KEY> -a "user@example.com" --no-input
jira issue edit <ISSUE-KEY> -l "new-label" --no-input                       # add label
jira issue edit <ISSUE-KEY> -l "-old-label" --no-input                      # remove label (prefix with -)
jira issue edit <ISSUE-KEY> --custom "customfield_10001=value" --no-input
```

### Transition / Move issue

```bash
jira issue move <ISSUE-KEY> "In Progress"
jira issue move <ISSUE-KEY> "Done" -R "Done"                                # with resolution
jira issue move <ISSUE-KEY> "In Review" --comment "Ready for review"
```

### Assign issue

```bash
jira issue assign <ISSUE-KEY> "$(jira me)"                                  # assign to self
jira issue assign <ISSUE-KEY> "user@example.com"                            # assign to user
jira issue assign <ISSUE-KEY> x                                             # unassign
```

### Add comment

```bash
jira issue comment add <ISSUE-KEY> "Comment text"
jira issue comment add <ISSUE-KEY> --internal "Internal note"               # internal comment
echo "multiline comment" | jira issue comment add <ISSUE-KEY>
```

### Clone issue

```bash
jira issue clone <ISSUE-KEY>
jira issue clone <ISSUE-KEY> -s "Cloned: new summary"
jira issue clone <ISSUE-KEY> -H "old-text:new-text"                         # replace text in clone
```

### Link issues

```bash
jira issue link <ISSUE-A> <ISSUE-B> "Blocks"
jira issue link <ISSUE-A> <ISSUE-B> "Duplicates"
jira issue link remote <ISSUE-KEY> "https://example.com" "Link title"
jira issue unlink <ISSUE-A> <ISSUE-B>
```

### Log work

```bash
jira issue worklog add <ISSUE-KEY> "2h 30m" --comment "Worked on X"
jira issue worklog add <ISSUE-KEY> "1d" --started "2026-07-15 09:00"
```

### Delete issue

```bash
jira issue delete <ISSUE-KEY>
jira issue delete <ISSUE-KEY> --cascade                                     # delete with subtasks
```
