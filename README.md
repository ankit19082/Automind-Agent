# AutoMind Agent 🧠

AutoMind is a Markdown-based autonomous DevOps AI agent. It monitors your Git repositories, analyzes code changes against Jira issues, updates ticket statuses, and notifies your team on Slack—all guided by instructions written in Markdown.

## 🚀 Features

- **Multi-Repo Monitoring**: Simultaneously track multiple projects.
- **Dynamic Discovery**: Add new repos just by dropping a file in `memory/repos/`.
- **Jira Correlation**: Automatically links commits to Jira tickets using AI and regex.
- **Smart Status Updates**: Moves tickets to "Move to QA" or "In Progress" based on implementation completeness.
- **Slack Notifications**: Sends professional, client-ready summaries of every change.
- **Rate Limit Resilience**: Automatically handles API rate limits with smart retry and recovery.

## 📁 Folder Structure

- `agents/`: Core agent definitions (see `automind.md` for the brain).
- `memory/`: Configuration and persistent state.
  - `repos/`: Individual repository configurations.
  - `config.md`: Global settings (Jira, Slack, Polling).
  - `state.md`: Sequential processing memory.
- `dashboard.html`: Visual status monitoring tool.

## 🛠️ Getting Started

### 1. Prerequisites

You need an AI CLI like **Claude Code** installed and authenticated.

### 2. Configuration

1. Create a `.env` file in the root directory:
   ```env
   JIRA_API_TOKEN=your_token
   SLACK_WEBHOOK_URL=your_webhook_url
   ```
2. Update `memory/config.md` with your Jira email and base URL.

### 3. Adding Repositories

Create a new file in `memory/repos/[repo-name].md`:

```markdown
# [Repository Name]

- `REPO_PATH`: `/path/to/your/local/repo`
- `REPO_BRANCH`: `dev`
```

### 4. Running the Agent

Execute the agent using the following command:

```bash
claude --dangerously-skip-permissions
```

The agent will automatically load `agents/automind.md` and begin the monitoring loop.

## 📊 Monitoring

Open `dashboard.html` in your browser to see a live view of all monitored repositories, their last processed commits, and recent activity.

## 🛠️ Troubleshooting

Logs are automatically generated in the `memory/` directory as `troubleshooting-[timestamp].md` if any errors occur.
