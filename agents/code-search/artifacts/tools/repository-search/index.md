---
name: repository_search_tool
description: NASA-verified public code repository discovery
mcp_server: CMR_MCP_Server
mcp_server_url_env: CODE_SEARCH_MCP_URL
mcp_auth_env: CODE_SEARCH_MCP_KEY
approval: never
affordances: [search, mcp]
step: 2
---

# repository_search_tool

Search the NASA-verified public code repository corpus.

## Purpose

Primary discovery channel for Step 2 of the reasoning strategy. The NASA-verified repository corpus is the agent's trusted starting point for finding code repositories — each repository has been curated for relevance and public accessibility.

## When the agent uses it

Always in Step 2, before any other search. The agent performs **at least two distinct queries** using:

1. The user's original query terms.
2. Synonym and related terms identified in Step 1.
3. Names of well-known codes from the Expected Codes checklist.
4. Broader category terms describing the class of software implied by the query.

If the initial query returns fewer than 6 candidates, or the agent's domain knowledge suggests well-known codes are missing, it issues additional targeted queries.

## Conceptual inputs

- `queries` — one or more search strings.
- Optional filters (language, domain category, etc.) as supported by the underlying search service.

## Conceptual outputs

A list of repository records. Per the agent's `RepositorySearchTool` upstream schema, each record contains:

- Repository name and URL
- GitHub-style metadata: stars, forks, repo age, commit frequency, maintenance status
- A reliability / signal score derived from metadata
- Summary of purpose from README / description
- Any additional signals the upstream tool surfaces

## Rules the agent follows

- Treat popularity signals (stars, forks) as supporting only — never decisive.
- Do not drop a candidate retained from this step later on, unless it is displaced in Step 7 ranking and then recorded in `excluded_candidates`.
- Record provenance as "NASA Repository Search" in the output.

## Upstream

Backed by `akd_ext/tools/code_search/repository_search.py`. Exposed via the `CMR_MCP_Server` hosted MCP (the label is reused from the CMR context — don't let it confuse you). See [`akd_ext/tools/`](https://github.com/NASA-IMPACT/akd-ext) in the upstream repo.
