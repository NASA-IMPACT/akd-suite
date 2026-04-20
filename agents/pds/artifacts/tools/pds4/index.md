---
name: pds4
description: PDS4-format discovery family — search and retrieval over the modern PDS4 archive
mcp_server: pds_mcp_server
mcp_server_url_env: PDS_MCP_URL
mcp_auth_env: PDS_MCP_KEY
approval: never
affordances: [search, catalog, mcp]
---

# PDS4 Tools (`pds4*`)

The PDS4 tool family is the agent's second broad-discovery surface (alongside PDS catalog). It targets the modern PDS4 archive structure: bundles, collections, products, instruments, instrument hosts, investigations, and targets.

## Allowed tools

- `pds4search_products_tool` — search for individual products.
- `pds4search_collections_tool` — search for collections.
- `pds4search_bundles_tool` — search for bundles.
- `pds4search_investigations_tool` — search for investigations (mission-like groupings).
- `pds4search_instruments_tool` — search for instruments.
- `pds4search_instrument_hosts_tool` — search for instrument hosts.
- `pds4search_targets_tool` — search for planetary targets.
- `pds4crawl_context_product_tool` — crawl context products (related lineage).
- `pds4get_product_tool` — retrieve a specific product.

## When the agent uses this family

- **Default entry point** for PDS4-era discovery. Per the catalog-first routing rule, `PDS4_MCP` is listed alongside `PDS_CATALOG_MCP` as the first place to search for broad queries.
- When the user's question maps cleanly onto a PDS4 hierarchy (bundle → collection → product).
- When checking whether a PDS3 result has a migrated PDS4 counterpart.
- For exact-identifier resolution when the identifier is a PDS4 `LID`, `LIDVID`, or URN.

## Conceptual inputs

Vary by tool, but commonly include:

- keyword / text query
- PDS4 `logical_identifier` (LID) or `LIDVID`
- mission / investigation / instrument filters
- target filters (body, region)
- temporal and spatial constraints

## Conceptual outputs

PDS4 records with:

- `logical_identifier`, `urn`, `version_id`
- `title`, `description`
- Parent context (bundle / collection / investigation)
- Instrument / target / mission metadata
- Access paths (stable archive URLs)

## Rules the agent follows

- Prefer PDS4 first in ranking when both PDS4 and PDS3 are available, but do not drop PDS3 if relevant.
- Use identifier-based retrieval (`pds4get_product_tool`, `pds4crawl_context_product_tool`) as minimal-call follow-ups, not primary discovery.
- Mark cross-version relationships as `equivalent`, `likely_related`, `legacy_predecessor`, `migrated_successor`, or `unknown` per the output format.

## Upstream

Exposed via the `pds_mcp_server` hosted MCP. Backs onto the PDS4-format archive services.
