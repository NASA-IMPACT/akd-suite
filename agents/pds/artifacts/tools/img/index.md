---
name: img
description: Planetary imaging node — image product search and facets
mcp_server: pds_mcp_server
mcp_server_url_env: PDS_MCP_URL
mcp_auth_env: PDS_MCP_KEY
approval: never
affordances: [search, node-specific, mcp]
---

# IMG Tools (`img_*`)

The IMG family is the node-specific tool set for the **IMG (Planetary Imaging)** node, covering image products across missions and instruments hosted there.

## Allowed tools

- `img_search_tool` — search image products at the IMG node.
- `img_count_tool` — count matches for an image search.
- `img_get_product_tool` — retrieve a specific image product record.
- `img_get_facets_tool` — retrieve facet values (e.g., available instruments, targets, processing levels).

## When the agent uses this family

Per the reasoning strategy's routing table, **IMG → `IMG_MCP`**. The agent calls IMG after catalog-first discovery identifies the query as IMG-node-appropriate, or when the user has explicitly asked for an imaging workflow or product-level image retrieval.

## Conceptual inputs

- Search: target, mission, instrument, processing level, time range, spatial bounds
- Count: same filter set, returns aggregate hit count
- Product retrieval: `PRODUCT_ID` or equivalent identifier
- Facets: optional filter scope

## Conceptual outputs

- Image product records with identifiers, instrument metadata, processing level, timestamps, and access URLs
- Facet value lists for building progressive filters
- Aggregate hit counts

## Rules the agent follows

- Narrow after catalog-first. IMG is not the starting point for broad discovery.
- Return both PDS4 and PDS3 versions of image products when both exist.
- Include missing metadata explicitly rather than inferring processing level or instrument.

## Upstream

Exposed via the `pds_mcp_server` hosted MCP. Backs onto IMG-node services.
