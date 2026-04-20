---
name: pds_catalog
description: Cross-node PDS catalog family — broad discovery across all PDS missions, targets, and datasets
mcp_server: pds_mcp_server
mcp_server_url_env: PDS_MCP_URL
mcp_auth_env: PDS_MCP_KEY
approval: never
affordances: [search, catalog, mcp]
---

# PDS Catalog Tools (`pds_catalog_*`)

The PDS catalog tool family is the agent's primary broad-discovery surface: cross-node search, mission listing, target listing, dataset resolution, and aggregate statistics. The agent's reasoning strategy explicitly names `PDS_CATALOG_MCP` as one of the two catalog-first entry points alongside PDS4.

## Allowed tools

- `pds_catalog_search_tool` — broad-keyword search across PDS holdings. Used first for most discovery-style queries.
- `pds_catalog_get_dataset_tool` — resolve a specific dataset (by identifier) and return its catalog record.
- `pds_catalog_list_missions_tool` — enumerate missions known to the catalog.
- `pds_catalog_list_targets_tool` — enumerate planetary targets and bodies.
- `pds_catalog_stats_tool` — aggregate counts / coverage statistics across the catalog.

## When the agent uses this family

- **Default entry point** for discovery queries. Per the catalog-first routing rule, the agent starts here (or with PDS4) and only narrows to node-specific tools afterward.
- When the user's query is broad (e.g., "find Mars surface mineralogy datasets") — search the catalog first, then narrow to ODE/GEO or IMG.
- When cross-checking whether a dataset has both PDS3 and PDS4 representations.
- When summarizing archive coverage for a user question ("what missions have data for Titan?").

## Conceptual inputs (per tool family)

- Search: keyword, mission, target, instrument, time range, entity level
- Resolution: dataset identifier (e.g., `DATA_SET_ID`, `PDS4 LID`)
- Listing: filter parameters (mission family, body type)
- Stats: aggregation scope

## Conceptual outputs

Structured records depending on the tool:

- Search → list of candidate datasets/bundles/collections with identifiers and summary metadata
- Get dataset → full catalog record (title, description, mission, target, instruments, archive lineage)
- List missions / targets → enumerated lists
- Stats → aggregate counts

## Rules the agent follows

- Use the catalog broadly, then narrow. Do not stack filters before trying a relaxed query.
- Always check for cross-version (PDS3 / PDS4) coverage when results appear.
- Record routing decisions in the output's Search Plan section.

## Upstream

Exposed via the `pds_mcp_server` hosted MCP. Backing data comes from NASA's PDS catalog. See the PDS agent's config in [`akd_ext/agents/pds_search_care.py`](https://github.com/NASA-IMPACT/akd-ext) for the live `allowed_tools` list.
