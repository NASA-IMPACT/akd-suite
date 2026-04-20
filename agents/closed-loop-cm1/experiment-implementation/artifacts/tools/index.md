# Experiment Implementation — Tools

Single hosted MCP server for submitting CM1 experiment batches.

## MCP server

- **Label**: `Job_Management_Server`
- **URL env var**: `EXPERIMENT_STATUS_MCP_URL` (no default; must be configured for this stage to function)
- **Auth env var**: `EXPERIMENT_STATUS_MCP_KEY`
- **Approval gating**: never
- **Server description**: MCP server for submitting CM1 experiment jobs to Temporal

## Allowed tools

- [`job-submit/`](./job-submit/) — submit the experiment batch.

If `EXPERIMENT_STATUS_MCP_URL` is not configured, the agent's tool list is empty and the submission phase is skipped.
