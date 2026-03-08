# AutoMind State

This file tracks the current state and persistent memory of the AutoMind agent, ensuring reliable recovery and sequential processing.

## State Structure
  - `last_processed_commit`: `b991925`
  - `last_run_time`: `2026-03-08T11:25:00Z`
  - `processed_issues`: `["SCRUM-9", "SCRUM-14", "SCRUM-15", "SCRUM-16", "SCRUM-17"]`

## Persistent Memory & Recovery
This file acts as the agent's persistent memory. In the event that the agent process stops or crashes, the state recorded here allows it to:
1. **Identify Gaps**: Determine which commits haven't been processed since the last successful run.
2. **Resume Processing**: Start exactly from the `last_processed_commit` to ensure no data is lost or skipped.
3. **Audit History**: Maintain a list of `processed_issues` to prevent duplicate processing.

## Sequential Processing Logic
To maintain data integrity, AutoMind processes commits sequentially:
1. **Fetch**: The agent fetches new commits from the repository since `last_processed_commit`.
2. **Order**: Commits are sorted chronologically from oldest to newest.
3. **Execute**: The agent processes each commit one-by-one.
4. **Commit State**: Only after a commit is successfully processed (and its associated Jira/Slack actions are complete) is `last_processed_commit` updated in this file.

## Responsibility
This file is the source of truth for the agent's operational progress. It must be updated atomically after each successful processing cycle to ensure robustness.
