---
name: job_status
description: Check the status of a CM1 experiment batch by job_id
mcp_server: Job_Management_Server
mcp_server_url_env: EXPERIMENT_STATUS_MCP_URL
mcp_auth_env: EXPERIMENT_STATUS_MCP_KEY
approval: never
affordances: [status, mcp]
---

# job_status

Check the completion state of a CM1 experiment batch.

## Purpose

MANDATORY Step 1 of the Research Report Generator's process. Before any other action, the agent must verify the batch represented by `job_id` has reached a terminal successful state. If not, the agent returns early with a status summary and never proceeds to figure fetching or report generation.

## When the agent uses it

Exactly once, at the very start of every run.

## Conceptual inputs

- `job_id` — the identifier returned by Stage-4's `job_submit`.

## Conceptual outputs

- Current status string. Terminal successful states treated as "finished": `finished`, `completed`, `done`, `success`.
- Any status outside that set halts the pipeline.

## Rules the agent follows

- Call once, never retry with different parameters.
- If status is not terminal-successful, stop immediately and return a `TextOutput` explaining current status.
- Do not fabricate a finished status in order to proceed.

## Upstream

Exposed via the `Job_Management_Server` hosted MCP.
