---
name: ads_search_tool
description: NASA ADS literature search for code discovery (Astrophysics only)
mcp_server: ADS_Search
mcp_server_url_env: ADS_SEARCH_MCP_URL
mcp_auth_env: CODE_SEARCH_ADS_SEARCH_KEY
approval: never
affordances: [search, literature, mcp]
step: 5
---

# ads_search_tool

Search NASA ADS (Astrophysics Data System) for papers that discuss, compare, or benchmark codes.

## Purpose

Step 5 of the reasoning strategy: **Astrophysics-only** literature-driven code discovery. ADS is a **discovery** channel — the agent dedicates all ADS queries to finding codes that may be missing from the candidate list, not to re-verifying codes already found.

## When the agent uses it

Only when:

- Step 1 classified the query as **Astrophysics**.
- The agent has compared the running candidate list against the Expected Codes checklist and identified priority gaps.

The agent skips this step entirely for non-Astrophysics queries.

## Conceptual inputs

ADS query strings using query operators like `abs:"..."` and `title:"..."`. Query patterns favored by the agent:

- Task term + software keywords: `abs:"<task_description>" AND (abs:"code" OR abs:"simulation" OR abs:"software")`
- Code comparison papers: `abs:"code comparison" AND abs:"<subfield_term>"` or `abs:"benchmark" AND abs:"<subfield_term>"`
- Review papers: `abs:"review" AND abs:"<task_term>" AND (abs:"code" OR abs:"software")`
- Named code: `title:"<code_name>"` or `abs:"<code_name>" AND abs:"<task_term>"`

Each query requests **at most 5 rows** and returns only `bibcode`, `title`, `year`, and `citation_count` fields — full abstracts are not requested (context-limit discipline).

## Conceptual outputs

A list of paper records with `bibcode`, `title`, `year`, `citation_count`. The agent extracts code names, GitHub URLs, and DOIs from those records.

## Rules the agent follows

- **Maximum 5 ADS queries per run.**
- Each query returns at most 5 rows and only the minimum fields needed.
- ADS-discovered candidates are added to the running candidate list and later ranked in Step 7. They do **not** need to also exist in the NASA corpus.
- Before including an ADS-discovered repository in the final output, verify its URL resolves to a public code repository.
- Citation counts from ADS results feed directly into Step 7 ranking as evidence of community adoption.
- Record provenance as "ADS" in the output.

## Upstream

Exposed via the `ADS_Search` hosted MCP (separate from the code-search MCP).
