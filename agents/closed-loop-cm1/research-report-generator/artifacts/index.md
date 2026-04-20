---
agent_id: research_report_generator
agent_class: ResearchReportGeneratorAgent
cm1_pipeline_stage: 5
source_repo: NASA-IMPACT/akd-ext
source_module: akd_ext/agents/closed_loop_cm1/research_report_generator.py
published_to_flow: true
published_note: INTERNAL ONLY — part of the CM1 specialized pipeline
---

# Research Report Generator — Artifact

Consumable knowledge layer for CM1 Stage 5. Derived strictly from the agent's current akd-ext system prompt and tool configuration.

## What's inside

- [`contexts/`](./contexts/) — role, objective, the 3-step PROCESS (mandatory job_status → job_plot → generate), the report structure spec, and the constraint set (scientific integrity, extraction rules, style, prohibitions).
- [`tools/`](./tools/) — the two MCP tools (`job_status`, `job_plot`) wired through `Job_Management_Server`.
