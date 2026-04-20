---
name: search_collections
description: Search the NASA CMR collection index and return candidate datasets matching the supplied query parameters.
mcp_server: CMR_MCP_Server
mcp_server_url_env: CMR_MCP_URL
approval: never
affordances: [search, mcp]
---

# search_collections

Search the NASA Common Metadata Repository collection index.

## Purpose

Given a set of query parameters (keyword, GCMD science keywords, platform, instrument, temporal range, spatial bounds, processing level, data center, etc.), return a list of candidate CMR collections with their metadata. This is the primary discovery tool for the CMR Agent — the step that converts a clarified user scope into dataset candidates.

## When the agent uses it

After the agent has:

1. Clarified the user's scope (variables, spatial bounds, temporal bounds),
2. Mapped terms to GCMD keywords,
3. Translated those keywords into CMR API parameters.

The agent calls `search_collections` with those parameters, inspects the returned collection metadata, and ranks results by metadata relevance. If results are insufficient, the agent may re-query with broader or differently shaped parameters, or enter the conditional multi-hop loop.

## Conceptual inputs

Parameters mirror those documented in the CMR Search API reference. Commonly used:

- `keyword` — free-text search over collection metadata
- `science_keywords` — GCMD hierarchy
- `platform` / `instrument` — satellite/instrument filters
- `temporal` — ISO 8601 range
- `spatial` / `bounding_box` — geographic filter
- `processing_level` — L0–L4
- `provider` / `data_center` — data center or archive

See [`../../contexts/cmr-api-reference.md`](../../contexts/cmr-api-reference.md) for the full parameter list.

## Conceptual outputs

A structured response containing:

- `hits` — total number of matching collections
- `items` — an array of collection records. Each record includes:
  - `concept_id` (collection ID starting with `C`)
  - `short_name`, `version_id`, `entry_title`
  - `platforms`, `instruments`
  - `temporal_extent`, `spatial_extent`
  - `processing_level`
  - Additional UMM fields when present

## Rules the agent follows when using this tool

- Do not recommend, select, or endorse the returned datasets.
- Preserve all metadata fields verbatim in the ranked output; mark missing fields as unknown rather than inferred.
- Log every call's endpoint, parameters, and paging behavior in the reproducibility section.
- Rank results by metadata relevance only; use usage signals only as a tie-breaker.

## Upstream

Wraps the CMR Search API `/search/collections` endpoint. Exposed to the agent through the `CMR_MCP_Server` hosted MCP. See [`akd_ext/agents/cmr_care.py`](https://github.com/NASA-IMPACT/akd-ext) for how the agent's current config wires it.
