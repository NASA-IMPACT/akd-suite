---
name: get_granules
description: Retrieve granules (individual data files) for a CMR collection, with optional temporal and spatial filtering.
mcp_server: CMR_MCP_Server
mcp_server_url_env: CMR_MCP_URL
approval: never
affordances: [search, mcp]
---

# get_granules

Retrieve individual granules belonging to a CMR collection.

## Purpose

Once the agent has identified one or more candidate collections via `search_collections`, `get_granules` returns the specific granule-level records for a collection. This gives the user file-level visibility (temporal coverage per granule, online availability, download paths).

## When the agent uses it

The CMR Agent is primarily a **collection-discovery** agent; it does not perform bulk granule retrieval. `get_granules` is used when the user asks for granule-level detail for a specific collection already surfaced in the ranked list, typically to support the user's manual verification step.

## Conceptual inputs

- `collection_concept_id` (required) — the CMR collection identifier (starts with `C`)
- `temporal` — ISO 8601 range for filtering granules
- `bounding_box` / `point` / `polygon` — spatial filter
- `online_only` / `downloadable` — availability filters
- `page_size` / `page_num` — pagination

## Conceptual outputs

A structured response containing:

- `hits` — total granule count matching the filter
- `items` — each granule with `concept_id` (starts with `G`), producer granule ID, temporal extent, spatial extent, and online access URLs (when present)

## Rules the agent follows when using this tool

- Does not download granule contents — link handoff only.
- Does not request or handle credentials for access-controlled granules.
- Logs the collection concept_id, filters applied, and paging behavior in the reproducibility section.

## Upstream

Wraps the CMR Search API `/search/granules` endpoint. Exposed via the `CMR_MCP_Server` hosted MCP.
