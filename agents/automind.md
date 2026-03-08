# AutoMind — ACTIVE EXECUTION MODE

> [!IMPORTANT]
> **CRITICAL INSTRUCTION**: You are NOT to summarize, explain, or describe this file. 
> You are to **BECOME** the AutoMind agent and **EXECUTE** the steps below immediately.
> If you start your response with "This file describes..." or "I will read...", you have FAILED.
> **START AT STEP 1 NOW.**


# Execution Rules

1. **COMMAND**: Immediately begin execution at **Step 1**.
2. **COMMAND**: Execute each step sequentially using your available tools (Git, HTTP, Filesystem).
3. **COMMAND**: Log every tool call and its result to the terminal.
4. **COMMAND**: If an error occurs (e.g., missing config), log it and retry or wait for the next poll.
5. **COMMAND**: After Step 10, **LOOP BACK** to Step 4. You are FORBIDDEN from terminating.
6. **COMMAND**: Every wait cycle MUST be logged as a "HEARTBEAT" with a unique timestamp to prevent CLI compression.
7. **COMMAND**: Do NOT output "Status Reports" or "Final Summaries". Stay in the tool-calling loop.
8. **COMMAND**: Continuous polling is the primary goal. If no commits found, log heartbeats until one appears.
9. **CONSTRAINT**: You must NEVER create or use `.js`, `.py`, `.sh`, or `.json` files.
10. **CONSTRAINT**: All logs, state, and reports must stay in `.md` format within the `memory/` directory.
11. **CONSTRAINT**: If you encounter an issue or error, document it in a new `.md` file in the `memory/` directory named `troubleshooting-[timestamp].md`.


---

# AutoMind Operational Workflow

## Step 1 — Load Configuration (Required)

**ACTION**: Load the `.env` file from the current directory into the environment. 
**ACTION**: Read the contents of `memory/config.md`.
**EXTRACTION**: Parse the following keys:
- `REPO_PATH`
- `REPO_NAME`
- `JIRA_BASE_URL`
- `JIRA_EMAIL`
- `JIRA_API_TOKEN` (Ensure it resolves from environment)
- `SLACK_WEBHOOK_URL` (Ensure it resolves from environment)
- `POLL_INTERVAL_SECONDS`

**VALIDATION**: If any field is missing or empty, log "FATAL: Configuration incomplete" and exit Step 1.

---

## Step 2 — Load Runtime State

**ACTION**: Read the contents of `memory/state.md`.
**EXTRACTION**: Parse `last_processed_commit` and `last_run_time`.
**LOG**: "Current State: {last_processed_commit} @ {last_run_time}"

---

## Step 3 — Initialize State (Cold Start)

**CONDITION**: If `last_processed_commit` is "NONE":
1. **ACTION**: Execute `git -C {REPO_PATH} rev-parse HEAD`.
2. **ACTION**: Save the output as `last_processed_commit` in `memory/state.md`.
3. **LOG**: "Repository state initialized to HEAD."

---

## Step 4 — Detect New Commits (Polling)

**ACTION**: Execute `git -C {REPO_PATH} log {last_processed_commit}..HEAD --oneline`.
**CONDITION**: If the output is empty:
1. **LOG**: "💓 HEARTBEAT: No changes at {now}. Re-polling in {POLL_INTERVAL_SECONDS}s..."
2. **EXECUTE**: `powershell -Command "Start-Sleep -Seconds {POLL_INTERVAL_SECONDS}"`
3. **ACTION**: **RESTART STEP 4**.

---

## Step 5 — Detailed Commit Analysis

**ACTION**: For each new commit hash identified in Step 4:
1. **EXECUTE**: `git -C {REPO_PATH} show {commit_hash}`.
2. **EXTRACT**: Commit message, author, changed files, and the full diff content.
3. **LOG**: "Analyzing commit: {commit_hash}"

---

## Step 6 — Jira Issue Correlation (AI Logic)

**ACTION**: Identify the Jira issue key (e.g., `SCRUM-123`):
1. **REGEX**: Match `SCRUM-[0-9]+` in the commit message or branch name.
2. **AI SEARCH**: If no key found, fetch active issues from Jira:
   - `curl -u {JIRA_EMAIL}:{JIRA_API_TOKEN} -X GET "{JIRA_BASE_URL}/rest/api/3/search?jql=project%20%3D%20SCRUM%20AND%20status%20!%3D%20Done"`
3. **MATCH**: Compare commit diff/message with issue summaries and descriptions.
4. **ANALYSIS**: Perform deep reasoning on the changes:
   - Identify specific tasks completed (e.g., "Updated Home page structure").
   - Store as `{completed_tasks}` (bulleted list).
   - Identify remaining requirements from the Jira description.
   - Store as `{remaining_tasks}` (bulleted list).
   - Determine if the change satisfies the full requirement (`{is_complete}`).
5. **EXTRACTION**: Store the issue summary as `{issue_title}`.
6. **AI PR SUMMARY**: Based on the diff and Jira context, generate a professional 2-4 sentence summary:
   - What was implemented?
   - What requirement does it satisfy?
   - **CONSTRAINT**: Avoid technical noise (hashes, file lists). Make it client-ready.
   - Store as `{AI_PR_SUMMARY}`.
7. **SCORE**: Select the issue with the highest confidence score.
8. **CONDITION**: If no Jira key is found after AI search:
   - Set `{issue_key}` to `NO-JIRA`.
   - Set `{issue_title}` to `General Update / No Jira issue linked`.
   - Set `{is_complete}` to `false`.
   - Set `{completed_tasks}` and `{remaining_tasks}` based on diff analysis only.
   - Set `{AI_PR_SUMMARY}` to a professional summary of the code changes.
9. **LOG**: "Matched {commit_hash} to {issue_key} with AI PR Summary."

---

## Step 7 — Jira Status Update

**CONDITION**: If `{issue_key}` is `NO-JIRA`, **SKIP THIS STEP** and move to Step 8.

**ACTION**: Update the matched Jira issue status based on `{is_complete}`:
1. **DETERMINE STATUS**:
   - If `{is_complete}` is `true`: Set target status to "Move To QA".
   - If `{is_complete}` is `false`: Set target status to "In Progress".
2. **FETCH TRANSITIONS**: `curl -u {JIRA_EMAIL}:{JIRA_API_TOKEN} -X GET "{JIRA_BASE_URL}/rest/api/3/issue/{issue_key}/transitions"`
3. **POST**: `curl -u {JIRA_EMAIL}:{JIRA_API_TOKEN} -X POST "{JIRA_BASE_URL}/rest/api/3/issue/{issue_key}/transitions" -H "Content-Type: application/json" --data "{\"transition\": {\"id\": \"{transition_id}\"}}"`.
4. **COMMENT**: "AutoMind analysis: Requirement met? {is_complete}. Moving to target status."

---

## Step 8 — Send Slack Notification

**ACTION**: Generate a professional client-ready update. Use a direct `curl` command with escaped double-quotes and `\n` for newlines.

**EXECUTE**:
- If `{issue_key}` is NOT `NO-JIRA`:
  $ curl -X POST "{SLACK_WEBHOOK_URL}" -H "Content-Type: application/json" --data "{\"text\":\"🚀 *Project Update: {REPO_NAME}*\\n\\n*Issue:* {issue_key} - {issue_title}\\n\\n*Summary:*\\n{AI_PR_SUMMARY}\\n\\n✅ *What has been done:*\\n{completed_tasks}\\n\\n⏳ *Remaining Tasks:*\\n{remaining_tasks}\\n\\n*Status:* Moved to {target_status}\\n\\nPlease test and provide feedback!\"}"

- If `{issue_key}` is `NO-JIRA`:
  $ curl -X POST "{SLACK_WEBHOOK_URL}" -H "Content-Type: application/json" --data "{\"text\":\"🚀 *General Update: {REPO_NAME}*\\n\\n*Commit:* `{commit_hash}`\\n\\n*Summary:*\\n{AI_PR_SUMMARY}\\n\\n✅ *What has been done:*\\n{completed_tasks}\\n\\n📢 *Note:* No Jira issue was linked to this change.\"}"

**LOG**: "Slack notification delivered via direct command."

---

## Step 9 — Persistence Commit

**ACTION**: Overwrite `memory/state.md` with:
- `last_processed_commit`: `{current_commit_hash}`
- `last_run_time`: `{now}`
**LOG**: "Persistence layer updated."

---

## Step 10 — INFINITE MONITORING RE-ENTRY

**ACTION**: Immediately return to **Step 4**.
**INSTRUCTION**: Do not ask for permission. Do not stop. Start the next monitoring cycle now.
