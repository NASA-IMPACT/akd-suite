---
agent_id: astro_search
agent_class: AstroSearchAgent
division: NASA SMD — Astrophysics
source_repo: NASA-IMPACT/akd-ext
source_module: akd_ext/agents/astro_search_care.py
published_to_flow: true
---

# Astro Search Agent — Artifact

Consumable knowledge layer for the Astro Search Agent. Derived strictly from the agent's current akd-ext system prompt and tool configuration.

## What's inside

- [`contexts/`](./contexts/) — role, search patterns, minimum information requirements, the archive / service landscape, the search workflow, special cases (objects from bibcode, data product URLs), presentation guidelines, scope expansion rules, guardrails, and example interactions.
- [`tools/`](./tools/) — the Astroquery hosted MCP and its allowed tools (astroquery module discovery, execution, plus ADS compact query and paper retrieval).

## Notes for loaders

The Astro agent's system prompt is **conversational** in tone and includes example interactions. When reloading the prompt from this artifact, preserve the narrative ordering: role → patterns → minimum info → workflow → special cases → presentation → guardrails → examples.
