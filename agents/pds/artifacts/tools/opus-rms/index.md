---
name: opus_rms
description: Outer Planets Unified Search (OPUS) — Ring-Moon Systems node
mcp_server: pds_mcp_server
mcp_server_url_env: PDS_MCP_URL
mcp_auth_env: PDS_MCP_KEY
approval: never
affordances: [search, node-specific, mcp]
---

# OPUS Tools (`opus_*`)

The OPUS family is the node-specific tool set for the **RMS (Ring-Moon Systems)** node, covering outer planet data, ring systems, and moon observations across missions like Cassini, Voyager, Galileo, and others served through OPUS.

## Allowed tools

- `opus_search_tool` — search OPUS holdings.
- `opus_count_tool` — count matches for a search.
- `opus_get_metadata_tool` — retrieve metadata for an OPUS observation.
- `opus_get_files_tool` — retrieve file listings for an observation.

## When the agent uses this family

Per the reasoning strategy's routing table, **RMS → `OPUS_MCP`**. The agent calls OPUS after catalog-first discovery identifies the query as RMS-node-appropriate (outer-planet targets, rings, icy moons), or when the user has explicitly targeted an OPUS workflow or provided an `OPUS_ID`.

## Conceptual inputs

- Search: target body, mission, instrument, observation time, geometry, surface region
- Metadata: OPUS observation identifier
- Files: OPUS observation identifier

## Conceptual outputs

- Observation records with identifiers, temporal/geometric metadata, associated product files, and access paths
- File listings
- Aggregate hit counts

## Rules the agent follows

- Catalog-first first. OPUS is a narrowing step.
- Exact-identifier lookups (`opus_get_metadata_tool`, `opus_get_files_tool`) are minimal-call follow-ups.
- Always check for PDS4 cross-version coverage since OPUS exposes both.

## Upstream

Exposed via the `pds_mcp_server` hosted MCP. Backs onto the OPUS service at the RMS node.
