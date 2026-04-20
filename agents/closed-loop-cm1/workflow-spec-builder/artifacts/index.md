---
agent_id: workflow_spec_builder
agent_class: WorkflowSpecBuilderAgent
cm1_pipeline_stage: 3
source_repo: NASA-IMPACT/akd-ext
source_module: akd_ext/agents/closed_loop_cm1/workflow_spec_builder.py
published_to_flow: true
published_note: INTERNAL ONLY — part of the CM1 specialized pipeline
---

# Workflow Spec Builder — Artifact

Consumable knowledge layer for CM1 Stage 3. Derived strictly from the agent's current akd-ext system prompt.

## What's inside

- [`contexts/`](./contexts/) — role, constraints (10 rules), the 9-step PROCESS, and the OUTPUT FORMAT spec (section order + matrix / summary conventions).
- [`embedded-contexts/`](./embedded-contexts/) — pointer to the embedded `cm1_readme.md` (sourced from upstream; not mirrored here due to size).

No `tools/` subdirectory — this agent has no configured tools.
