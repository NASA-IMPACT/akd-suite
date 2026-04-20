---
name: job_submit
description: Submit a CM1 experiment batch (list of ExperimentSpec + workspace name + base template) and receive a job_id
mcp_server: Job_Management_Server
mcp_server_url_env: EXPERIMENT_STATUS_MCP_URL
mcp_auth_env: EXPERIMENT_STATUS_MCP_KEY
approval: never
affordances: [submit, mcp]
---

# job_submit

Submit the experiment batch described by Stage-4's structured output and receive a `job_id` identifying the batch.

## Purpose

The only MCP tool Stage 4 calls. It accepts the agent's `experiments` / `workspace_name` / `base_template` payload and hands off to the downstream Temporal-based Python engine that actually builds the workspace on disk and submits the SLURM jobs.

## When the agent uses it

Once, after constructing all `ExperimentSpec` objects and validating internal consistency (baseline inheritance, feasibility flags, exact parameter names). The return value (`job_id`) is required in the agent's output — without it the run fails.

## Conceptual inputs

- `experiments` — list of `ExperimentSpec` objects.
- `workspace_name` — descriptive workspace directory name.
- `base_template` — CM1 case template directory name.

## Conceptual outputs

- `job_id` — identifier for the submitted batch. Used by Stage 5 (`job_status`, `job_plot`) to track completion and fetch figures.

## Rules the agent follows

- Must call `job_submit` exactly once.
- Must capture the returned `job_id` and surface it in the output's `job_id` field.
- Must produce a markdown implementation report summarising what was submitted.

## Upstream

Exposed via the `Job_Management_Server` hosted MCP. The agent's config only wires this tool when `EXPERIMENT_STATUS_MCP_URL` is set.
