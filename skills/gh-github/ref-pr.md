# Pull Request Commands

## List PRs

```bash
gh pr list                                          # open PRs in current repo
gh pr list -s closed -L 10                          # last 10 closed PRs
gh pr list -s merged --search "author:@me"          # your merged PRs
gh pr list -A <author> -s all                       # all PRs by author
gh pr list -B main                                  # PRs targeting main
gh pr list -l "bug" -l "priority/high"              # PRs with specific labels
gh pr list -R <owner>/<repo>                        # PRs in another repo
```

### Useful JSON fields for `--json`

`number`, `title`, `state`, `author`, `baseRefName`, `headRefName`, `createdAt`, `updatedAt`, `mergedAt`, `closedAt`, `additions`, `deletions`, `changedFiles`, `labels`, `reviewDecision`, `statusCheckRollup`, `isDraft`, `url`, `body`, `comments`, `reviews`, `commits`, `files`, `assignees`, `milestone`

```bash
gh pr list --json number,title,author,reviewDecision,updatedAt -L 50
gh pr list --json number,title,state --jq '.[] | "\(.number)\t\(.title)\t\(.state)"'
```

## View PR details

```bash
gh pr view <number>                                 # rich text view
gh pr view <number> --comments                      # include comments
gh pr view <number> --json body,comments,reviews    # structured JSON
gh pr view <number> -w                              # open in browser
```

## View PR diff and CI checks

```bash
gh pr diff <number>                                 # full diff
gh pr diff <number> --name-only                     # changed file names only
gh pr checks <number>                               # CI check status
gh pr checks <number> --watch                       # watch CI progress
gh pr checks <number> --json name,state,conclusion  # structured check results
```

## View PR review comments

```bash
gh api repos/<owner>/<repo>/pulls/<number>/comments --jq '.[] | "[\(.path):\(.line)] \(.user.login): \(.body)"'
gh api repos/<owner>/<repo>/pulls/<number>/reviews --jq '.[] | "\(.user.login): \(.state) - \(.body)"'
```

## Write Operations (require user permission)

### Create a PR

```bash
gh pr create --title "<title>" --body "<body>"
gh pr create --title "<title>" --body "<body>" -B main -l "enhancement"
gh pr create --fill                                 # fill from commit messages
gh pr create --draft                                # create as draft
```

### Comment on a PR

```bash
gh pr comment <number> --body "comment text"
```

### Review a PR

```bash
gh pr review <number> --approve
gh pr review <number> --approve --body "LGTM"
gh pr review <number> --comment --body "feedback"
gh pr review <number> --request-changes --body "please fix X"
```

### Merge a PR

```bash
gh pr merge <number> --squash --delete-branch
gh pr merge <number> --rebase
gh pr merge <number> --merge
gh pr merge <number> --auto --squash               # auto-merge when checks pass
```

### Edit PR metadata

```bash
gh pr edit <number> --title "new title"
gh pr edit <number> --add-label "ready" --remove-label "wip"
gh pr edit <number> --add-reviewer <user>
gh pr edit <number> --add-assignee @me
```

### Close / Reopen

```bash
gh pr close <number>
gh pr reopen <number>
```
