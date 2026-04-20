---
agent_id: code_search_care
agent_class: CodeSearchCareAgent
division: cross-SMD (Astrophysics, BPS, Earth, Heliophysics, Planetary)
source_repo: NASA-IMPACT/akd-ext
source_module: akd_ext/agents/code_search_care.py
published_to_flow: true
---

# Code Search Agent — Artifact

Consumable knowledge layer for the Code Search CARE Agent. Derived strictly from the agent's current akd-ext system prompt and tool configuration.

## What's inside

- [`contexts/`](./contexts/) — role, operational constraints, the multi-pass PROCESS (Steps 1–8), and the OUTPUT FORMAT contract.
- [`tools/`](./tools/) — the three authoritative data sources the agent uses: NASA-verified repository corpus + SDE + code-signals (via one MCP server); ADS literature (second MCP server); and web search (supplementary).

## Notes for loaders

The Code Search agent has a strict sequence discipline: Steps 1–8 must be executed in order, no step may be skipped, and the candidate list is cumulative during Steps 2–6 with ranking occurring only in Step 7. See [`contexts/reasoning-strategy.md`](./contexts/reasoning-strategy.md).
