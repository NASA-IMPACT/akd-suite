---
agent_id: capability_feasibility_mapper
agent_class: CapabilityFeasibilityMapperAgent
division: Cross-SMD (atmospheric simulation)
cm1_pipeline_stage: 2
source_repo: NASA-IMPACT/akd-ext
source_module: akd_ext/agents/closed_loop_cm1/capability_feasibility_mapper.py
published_to_flow: true
published_note: INTERNAL ONLY — part of the CM1 specialized pipeline
---

# Capability & Feasibility Mapper — Artifact

Consumable knowledge layer for CM1 Stage 2. Derived strictly from the agent's current akd-ext system prompt and embedded context documents.

## What's inside

- [`contexts/`](./contexts/) — role, objective, evidence rules, human decision boundary, the ten-step PROCESS, and the output format spec.
- [`embedded-contexts/`](./embedded-contexts/) — the `cm1_readme.md` and `cluster_it.md` documents that are injected into the agent's system prompt at construction time.

No `tools/` subdirectory — this agent has no configured tools.
