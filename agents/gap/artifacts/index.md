---
agent_id: gap_agent
agent_class: GapAgent
division: cross-SMD
cm1_pipeline_stage: 1
source_repo: NASA-IMPACT/akd-ext
source_module: akd_ext/agents/gap.py
published_to_flow: true
published_note: INTERNAL ONLY (part of the CM1 pipeline; flagged as "do not select in planner workflows")
---

# Gap Agent — Artifact

Consumable knowledge layer for the Gap Agent. Derived strictly from the agent's current akd-ext system prompt.

## What's inside

- [`contexts/`](./contexts/) — role, context and inputs, constraints (epistemic + transparency + HITL authority), the six-stage PROCESS, and the output format.

No `tools/` subdirectory — this agent has no configured tools. It operates purely over LLM reasoning against the user-provided corpus.
