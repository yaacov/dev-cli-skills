# dev-cli-skills

AI agent skills for the **GitHub CLI** (`gh`), **Jira CLI** (`jira`), and the **Forklift/MTV CLI** (`oc mtv`). Works with [Cursor](https://www.cursor.com/) and [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## What Can I Do with These Skills?

Just open a chat and ask:

| Ask the agent to… | Skill used |
|--------------------|------------|
| *"Show me my open PRs and their CI status"* | **gh-github** |
| *"List issues assigned to me in PROJECT"* | **jira-issues** |
| *"Search GitHub code for `handleMigration` across our org"* | **gh-github** |
| *"What did I work on last week? Show PRs and Jira tickets"* | **gh-github** + **jira-issues** |
| *"Show the current sprint status"* | **jira-issues** |
| *"Find all open bugs in PROJECT created this month"* | **jira-issues** |
| *"Run the manual test flow for the Forklift feature I just built"* | **mtv-dev** |

## Skills

| Skill | Description |
|-------|-------------|
| [gh-github](skills/gh-github/SKILL.md) | List PRs, view issues, search code, check CI status, review GitHub activity |
| [jira-issues](skills/jira-issues/SKILL.md) | List issues, view tickets, check sprints, search with JQL, manage epics |
| [mtv-dev](skills/mtv-dev/SKILL.md) | Run the manual Forklift/MTV feature test flow: namespace, vSphere provider from `GOVC_*`, migration plan |

## Quick Start

### Cursor / Claude Code

```bash
curl -sSL https://raw.githubusercontent.com/yaacov/dev-cli-skills/main/install.sh | bash
```

The script clones the repo to `~/.local/share/dev-cli-skills` (or pulls if
already present) and creates user-wide symlinks in `~/.cursor/skills` and/or
`~/.claude/skills` depending on which directories exist.

Run the same command again any time to **update**.

Or clone and run manually:

```bash
git clone https://github.com/yaacov/dev-cli-skills.git ~/.local/share/dev-cli-skills
bash ~/.local/share/dev-cli-skills/install.sh
```

### Claude Code Plugin (alternative)

```bash
claude plugin marketplace add yaacov/dev-cli-skills
claude plugin install dev-cli-skills@yaacov
```

To update later:

```bash
claude plugin marketplace update yaacov
claude plugin install dev-cli-skills@yaacov
```

To uninstall:

```bash
claude plugin uninstall dev-cli-skills@yaacov
```

## Removal

### Symlinks

```bash
for skill in gh-github jira-issues mtv-dev; do
  rm -f ~/.cursor/skills/"$skill"
  rm -f ~/.claude/skills/"$skill"
done
```

### Cloned Repository

```bash
rm -rf ~/.local/share/dev-cli-skills
```

## Prerequisites

### GitHub CLI

```bash
brew install gh
gh auth login
```

### Jira CLI

```bash
go install github.com/ankitpokhrel/jira-cli/cmd/jira@latest
jira init
```

### MTV CLI

```bash
oc mtv --help   # kubectl-mtv plugin
```

## Design Principles

- **Read-only by default** — write operations always require explicit user permission
- **Self-learning** — skills instruct the agent to run `--help` for unfamiliar commands
- **No secrets** — all examples use generic placeholders (`<owner>/<repo>`, `<PROJECT>`, etc.)

## License

[Apache-2.0](LICENSE)
