# Research Report Generator — Tools

Single hosted MCP server with two allowed tools for checking job completion and fetching result figures.

## MCP server

- **Label**: `Job_Management_Server`
- **URL env var**: `EXPERIMENT_STATUS_MCP_URL`
- **Auth env var**: `EXPERIMENT_STATUS_MCP_KEY`
- **Approval gating**: never
- **Server description**: MCP server for checking CM1 experiment job status and fetching result figures

## Allowed tools

- [`job-status/`](./job-status/) — verify batch completion (MANDATORY Step 1).
- [`job-plot/`](./job-plot/) — fetch figure URLs after confirmation (Step 2).

## Notes

- `job_submit` (used by Stage 4) is not whitelisted here. Stage 5 only checks status and retrieves plots.
- Both tools must be called exactly once per run (one `job_status`, one `job_plot`).
