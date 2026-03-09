# AutoMind State

This file tracks the current state and persistent memory of the AutoMind agent, ensuring reliable recovery and sequential processing.

## State Structure

- `HK-Booking-Flow_LAST_PROCESSED_COMMIT`: `75a22413ac6d9bfb8577471fb57b42e51bdd62ba`
- `HK-Booking-Flow_LAST_RUN_TIME`: `2026-03-09T15:31:03Z`

- `processed_issues`: `["SCRUM-21", "SCRUM-20"]`

- `my-app_LAST_PROCESSED_COMMIT`: `428adb7bd67d479e09adb6e5ebb1a0dde4073e3d`
- `my-app_LAST_RUN_TIME`: `2026-03-09T15:30:40Z`

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
