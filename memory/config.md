# AutoMind Configuration

This file contains the configuration parameters for the AutoMind agent. Sensitive tokens should be provided via environment variables as per the instructions below.

## Jira Configuration

- `JIRA_BASE_URL`: `https://ankitgroovy19.atlassian.net`
- `JIRA_EMAIL`: `ankitgroovy19@gmail.com`
- `JIRA_API_TOKEN`: `${ENV:JIRA_API_TOKEN}`
- `JIRA_PROJECT_KEY`: `SCRUM`

## Slack Configuration

- `SLACK_WEBHOOK_URL`: `${ENV:SLACK_WEBHOOK_URL}`

## Repository Discovery

Repositories are automatically discovered from the `memory/repos/` directory. To add a new repository, create a new `.md` file in that directory with the following structure:

```markdown
# [Repository Name]

- `REPO_PATH`: `/path/to/repo`
- `REPO_BRANCH`: `branch-name`
```

## Runtime Configuration

- `POLL_INTERVAL_SECONDS`: `300`

## Responsibility

This file is responsible for defining how AutoMind behaves and what resources it can access. It allows users to customize the agent's constraints and capabilities. For security, never hardcode actual tokens in this file; use environment variables instead.
