---
agent_id: pds_search
agent_class: PDSSearchAgent
division: NASA SMD — Planetary Science
source_repo: NASA-IMPACT/akd-ext
source_module: akd_ext/agents/pds_search_care.py
published_to_flow: true
---

# PDS Agent — Artifact

Consumable knowledge layer for the PDS Search Agent. Derived from the agent's current akd-ext system prompt and tool configuration.

## What's inside

- [`contexts/`](./contexts/) — the agent's role, scope, constraints, reasoning strategy (search rules + default workflow), and output format.
- [`tools/`](./tools/) — the tool families wired through the `pds_mcp_server` hosted MCP. All 30 allowed tools are grouped by service family (PDS catalog, PDS4, ODE for GEO/Mars, OPUS for RMS, IMG for imaging, SBN for small bodies).

## Notes for loaders

Primary prompt sections in order: `role-and-objective.md`, `scope.md`, `constraints.md`, `reasoning-strategy.md`, `output-format.md`. The PDS agent uses two output templates — Template A (structured primary) and Template D (hard stop) — both preserved in `output-format.md`.
