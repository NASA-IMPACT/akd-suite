---
agent_id: cmr_care
agent_class: CMRCareAgent
division: NASA SMD — Earth Science
source_repo: NASA-IMPACT/akd-ext
source_module: akd_ext/agents/cmr_care.py
published_to_flow: true
---

# CMR Agent — Artifact

This artifact is the consumable knowledge layer for the CMR CARE Agent. Content is derived strictly from the agent's current akd-ext system prompt and tool configuration.

## What's inside

- [`contexts/`](./contexts/) — the agent's role, reasoning strategy, constraints, output format, and the embedded CMR Search API reference material that ships with the prompt.
- [`tools/`](./tools/) — the tools the agent is configured to use.

## Notes for loaders

An artifact loader (for example a future akd-ext component that re-hydrates this directory into a live agent's context) should treat `contexts/role-and-objective.md`, `contexts/reasoning-strategy.md`, `contexts/constraints.md`, and `contexts/output-format.md` as the four primary sections of the system prompt, in that order. `contexts/cmr-api-reference.md` is the supplementary reference appended to the prompt under the agent's ADDITIONAL CONTEXT heading.
