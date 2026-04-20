---
name: sbn
description: Small Bodies Node — comets, asteroids, KBOs, TNOs; search by object or coordinate
mcp_server: pds_mcp_server
mcp_server_url_env: PDS_MCP_URL
mcp_auth_env: PDS_MCP_KEY
approval: never
affordances: [search, node-specific, mcp]
---

# SBN Tools (`sbn_*`)

The SBN family is the node-specific tool set for the **Small Bodies Node**, covering comets, asteroids, Kuiper Belt objects, and related small-body observations.

## Allowed tools

- `sbn_search_object_tool` — search by named small body (comet, asteroid, KBO).
- `sbn_search_coordinates_tool` — search by sky coordinates or target position.
- `sbn_list_sources_tool` — enumerate data sources available at the SBN node.

## When the agent uses this family

Per the reasoning strategy's routing table, **SBN → `SBN_MCP`**. The agent calls SBN after catalog-first discovery identifies the query as SBN-node-appropriate, or when the user has explicitly asked about a named small body or provided target coordinates.

## Conceptual inputs

- Object search: small-body name or identifier
- Coordinate search: RA/Dec or heliocentric position, search radius
- Source listing: filter scope (optional)

## Conceptual outputs

- Small-body observation records with identifiers, instrument/mission context, and access URLs
- Source enumeration with descriptions
- Geometry and metadata consistent with PDS small-body conventions

## Rules the agent follows

- Narrow after catalog-first. SBN is not the starting point for a broad query.
- When a body has been observed by multiple missions, return the candidates rather than picking one.
- Include missing metadata explicitly.

## Upstream

Exposed via the `pds_mcp_server` hosted MCP. Backs onto SBN-node services.
