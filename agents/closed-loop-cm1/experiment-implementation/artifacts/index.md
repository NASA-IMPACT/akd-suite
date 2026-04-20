---
agent_id: experiment_implementation
agent_class: ExperimentImplementationAgent
cm1_pipeline_stage: 4
source_repo: NASA-IMPACT/akd-ext
source_module: akd_ext/agents/closed_loop_cm1/experiment_implementation.py
published_to_flow: true
published_note: INTERNAL ONLY — part of the CM1 specialized pipeline
---

# Experiment Implementation — Artifact

Consumable knowledge layer for CM1 Stage 4. Derived strictly from the agent's current akd-ext system prompt, config, and tool configuration.

## What's inside

- [`contexts/`](./contexts/) — role, objective, the ten CRITICAL RULES, the 5-step PROCESS, and the OUTPUT FORMAT spec. Also includes a description of the `FileEdit` / `ExperimentSpec` data model the agent emits.
- [`tools/`](./tools/) — the `job_submit` MCP tool used to submit the experiment batch.
- [`embedded-contexts/`](./embedded-contexts/) — pointer to the embedded `cm1_readme.md`.
