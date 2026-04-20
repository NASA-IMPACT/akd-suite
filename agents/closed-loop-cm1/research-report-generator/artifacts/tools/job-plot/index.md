---
name: job_plot
description: Fetch figure/plot URLs for a completed CM1 experiment batch
mcp_server: Job_Management_Server
mcp_server_url_env: EXPERIMENT_STATUS_MCP_URL
mcp_auth_env: EXPERIMENT_STATUS_MCP_KEY
approval: never
affordances: [retrieve, mcp]
---

# job_plot

Fetch the figure / plot URLs produced by a completed experiment batch.

## Purpose

Step 2 of the Research Report Generator's process. After `job_status` confirms completion, the agent calls `job_plot` once to retrieve all figure URLs generated across the experiments in the batch. These URLs are embedded in the report using markdown image syntax.

## When the agent uses it

Once, immediately after `job_status` confirms a terminal successful state. Never before.

## Conceptual inputs

- `job_id` — the identifier returned by Stage-4's `job_submit`.

## Conceptual outputs

- List of figure URLs spanning all experiments in the batch. Each URL corresponds to a plot the Python engine generated from a completed experiment.
- If no figures are returned, the agent continues without figures but notes this in the report.

## Rules the agent follows

- Call once.
- Every URL returned MUST be embedded in the report as a markdown image.
- Filenames derived from URLs are used to reference figures in the Results section.

## Upstream

Exposed via the `Job_Management_Server` hosted MCP.
