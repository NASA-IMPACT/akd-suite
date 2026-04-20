---
name: get_collection_metadata
description: Fetch full metadata for a specific CMR collection by concept ID.
mcp_server: CMR_MCP_Server
mcp_server_url_env: CMR_MCP_URL
approval: never
affordances: [lookup, mcp]
---

# get_collection_metadata

Fetch the full metadata record for a single CMR collection.

## Purpose

When the agent needs richer detail about a specific collection than the summary returned from `search_collections` — for example, to populate the fact-check / user verification list with full variable definitions, processing-level notes, platform details, or publication DOIs — it calls `get_collection_metadata` with the collection's concept ID.

## When the agent uses it

After `search_collections` has surfaced a candidate collection and the agent is preparing the ranked list or the fact-check section for output. Not called on every collection; only where summary metadata is ambiguous or the user asked for detail on a specific collection.

## Conceptual inputs

- `concept_id` (required) — the collection concept ID (starts with `C`, e.g., `C123456-LPDAAC_ECS`)

Optional format controls (response format, field selection) mirror the CMR Search API conventions documented in the reference material.

## Conceptual outputs

A full UMM collection record:

- Identifiers: `concept_id`, `short_name`, `version`, `entry_title`
- Platforms, instruments
- Temporal extents, spatial extents
- Processing level
- Data dates (creation, revisions)
- Abstract / description
- Related URLs (documentation, data access)
- Provider and archive center
- Any additional UMM fields

## Rules the agent follows when using this tool

- Preserves all metadata fields verbatim; does not paraphrase or infer.
- Missing fields are explicitly listed in the output under "missing or ambiguous metadata," never silently filled.
- Logs the concept_id queried and timestamp in the reproducibility section.

## Upstream

Wraps CMR Search API lookups for a specific collection (scoped by concept_id). Exposed via the `CMR_MCP_Server` hosted MCP.
