# CMR Agent

The **CMR CARE Agent** is an AKD agent for discovering NASA Earthdata datasets via the Common Metadata Repository (CMR). It uses the **CARE** reasoning loop — Clarify, Analyze, Rank, Explain — to guide a user from an Earth-science question to a curated, ranked, metadata-backed list of candidate collections.

## At a glance

| | |
| --- | --- |
| **Purpose** | Help users discover, organize, and understand NASA Earthdata CMR datasets relevant to an Earth-science question |
| **Users** | Master's-level and above Earth science researchers |
| **Data source** | NASA Common Metadata Repository (CMR), GCMD KMS, Semantic Scholar (optional, indirect discovery) |
| **Division** | NASA SMD — Earth Science |
| **Reasoning pattern** | CARE loop with clarify → interpret → search → rank → explain; conditional multi-hop for indirect discovery |
| **Output** | Structured sections: clarifying questions, interpreted scope, ranked dataset list, reproducibility log, fact-check/verification list |
| **Source** | [NASA-IMPACT/akd-ext — `akd_ext/agents/cmr_care.py`](https://github.com/NASA-IMPACT/akd-ext) |

## What a successful run looks like

The user asks an Earth-science question. The agent:

1. Asks clarifying questions about variables, spatial bounds, temporal bounds, and whether indirect inference is permitted.
2. Maps the clarified scope to GCMD keywords and CMR API parameters.
3. Queries the CMR collection index.
4. Ranks returned collections by **metadata relevance only** — no claims about scientific suitability.
5. Explains why each dataset appears and what is missing.
6. Provides a full reproducibility log (endpoints, parameters, GCMD mappings, timestamps) and a list of items the user should verify manually.

The agent is explicitly **non-prescriptive**. It never recommends, endorses, or selects a dataset. It surfaces candidates and metadata; the researcher decides.

## What it never does

- Recommends, selects, or endorses datasets
- Claims quality, accuracy, or suitability
- Infers or fabricates missing metadata
- Executes searches without explicit user confirmation
- Performs downloads, requests credentials, or interacts with carts
- Operates outside Earth science

## Deeper reading

- [`artifacts/`](./artifacts/) — the full knowledge layer (role, reasoning strategy, constraints, output format, CMR API reference, tool definitions).
- Upstream implementation: [`akd_ext/agents/cmr_care.py`](https://github.com/NASA-IMPACT/akd-ext) in the akd-ext repo.
- Related: [`agents/code-search/`](../code-search/) (same CARE pattern), [`agents/pds/`](../pds/) (sister planetary agent), [`docs/mcp.md`](../../docs/mcp.md) (MCP integration used by this agent).
