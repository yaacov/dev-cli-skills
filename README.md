# dev-cli-skills

Claude Code plugin providing AI agent skills for the **GitHub CLI** (`gh`) and **Jira CLI** (`jira`).

## Skills

| Skill | Description |
|-------|-------------|
| [gh-github](skills/gh-github/SKILL.md) | List PRs, view issues, search code, check CI status, review GitHub activity |
| [jira-issues](skills/jira-issues/SKILL.md) | List issues, view tickets, check sprints, search with JQL, manage epics |

## Installation

Requires [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI.

```bash
# Add the marketplace
claude plugin marketplace add yaacov/dev-cli-skills

# Install the plugin
claude plugin install dev-cli-skills@yaacov
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

## Design Principles

- **Read-only by default** — write operations always require explicit user permission
- **Self-learning** — skills instruct the agent to run `--help` for unfamiliar commands
- **No secrets** — all examples use generic placeholders (`<owner>/<repo>`, `<PROJECT>`, etc.)

## License

Apache-2.0
