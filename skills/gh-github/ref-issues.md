# Issue Commands

## List issues

```bash
gh issue list                                       # open issues in current repo
gh issue list -s all -L 30                          # all issues, limit 30
gh issue list -a @me                                # assigned to you
gh issue list -A <author>                           # created by author
gh issue list -l "bug"                              # with specific label
gh issue list -m "v2.12"                            # in milestone
gh issue list -s closed -L 10                       # last 10 closed
gh issue list -R <owner>/<repo>                     # in another repo
```

### Useful JSON fields for `--json`

`number`, `title`, `state`, `stateReason`, `author`, `assignees`, `labels`, `milestone`, `createdAt`, `updatedAt`, `closedAt`, `body`, `comments`, `url`, `isPinned`, `projectItems`

```bash
gh issue list --json number,title,state,labels,assignees -L 50
gh issue list --json number,title,updatedAt --jq '.[] | "\(.number)\t\(.title)"'
```

## View issue details

```bash
gh issue view <number>                              # rich text view
gh issue view <number> --comments                   # include comments
gh issue view <number> --json body,comments         # structured JSON
gh issue view <number> -w                           # open in browser
```

## Issue status

```bash
gh issue status                                     # issues relevant to you
```

## Write Operations (require user permission)

### Create an issue

```bash
gh issue create --title "<title>" --body "<body>"
gh issue create --title "<title>" --body "<body>" -l "bug" -a @me
gh issue create --title "<title>" --body-file <path>
```

### Comment on an issue

```bash
gh issue comment <number> --body "comment text"
```

### Edit issue metadata

```bash
gh issue edit <number> --title "new title"
gh issue edit <number> --body "updated body"
gh issue edit <number> --add-label "priority/high" --remove-label "triage"
gh issue edit <number> --add-assignee <user>
gh issue edit <number> --milestone "v2.12"
```

### Close / Reopen

```bash
gh issue close <number>
gh issue close <number> -r "completed"              # with reason
gh issue reopen <number>
```

### Transfer / Delete

```bash
gh issue transfer <number> <destination-repo>
gh issue delete <number>
```
