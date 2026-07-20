# Search and API Commands

## Search pull requests

```bash
gh search prs "query terms"
gh search prs --author=<username> --state=open
gh search prs --repo=<owner>/<repo> --merged-at=">2026-04-20"
gh search prs --reviewed-by=<username> --state=closed -L 20
gh search prs --involves=<username> --updated=">2026-06-01"
```

## Search issues

```bash
gh search issues "query terms"
gh search issues --repo=<owner>/<repo> --state=open
gh search issues --assignee=<username> --label="bug"
gh search issues --author=<username> --created=">2026-04-20" -L 50
```

## Search code

```bash
gh search code "functionName" --repo=<owner>/<repo>
gh search code "import something" --language=go
gh search code "TODO" --repo=<owner>/<repo> --filename="*.go"
```

## Search commits

```bash
gh search commits "fix bug" --repo=<owner>/<repo>
gh search commits --author=<username> --committer-date=">2026-04-20"
```

## Search repos

```bash
gh search repos "keyword" --language=go --sort=stars
gh search repos --owner=<org> --topic="kubernetes"
```

## GitHub API (gh api)

For any GitHub API endpoint not covered by dedicated commands. Read operations only unless user grants permission.

### REST API

```bash
gh api repos/<owner>/<repo>                                         # repo info
gh api repos/<owner>/<repo>/contributors --jq '.[].login'          # contributors
gh api repos/<owner>/<repo>/pulls/<number>/comments                 # PR review comments
gh api repos/<owner>/<repo>/pulls/<number>/reviews                  # PR reviews
gh api repos/<owner>/<repo>/actions/runs --jq '.workflow_runs[:5] | .[].name'  # recent CI runs
gh api user                                                         # authenticated user
gh api user/repos --paginate --jq '.[].full_name'                  # all your repos
```

### Useful patterns

Placeholders `{owner}` and `{repo}` auto-fill from the current git repo:

```bash
gh api repos/{owner}/{repo}/pulls --jq '.[].title'
```

Pagination:

```bash
gh api repos/<owner>/<repo>/issues --paginate --jq '.[].title'
```

Filtering with jq:

```bash
gh api repos/<owner>/<repo>/pulls --jq '[.[] | select(.user.login == "<username>")] | length'
```

### Write operations via API (require user permission)

```bash
gh api repos/<owner>/<repo>/issues -f title="Title" -f body="Body"           # POST: create issue
gh api repos/<owner>/<repo>/issues/<number> -X PATCH -f state="closed"       # PATCH: close issue
gh api repos/<owner>/<repo>/issues/<number>/comments -f body="comment"       # POST: add comment
```
