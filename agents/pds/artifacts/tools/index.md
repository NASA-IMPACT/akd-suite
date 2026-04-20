# PDS Agent — Tools

The PDS Search Agent wires a single hosted MCP server, `pds_mcp_server`, with 30 allowed tools spanning six service families. The agent's default workflow is **catalog-first**: start broad with PDS catalog and PDS4 family tools, then narrow with node-specific families.

## MCP server

- **Label**: `pds_mcp_server`
- **URL env var**: `PDS_MCP_URL`
- **Auth env var**: `PDS_MCP_KEY`
- **Approval gating**: never

## Tool families

| Family | Directory | What it covers | When the agent uses it |
| --- | --- | --- | --- |
| PDS catalog | [`pds-catalog/`](./pds-catalog/) | Cross-node catalog search, mission and target lists, dataset resolution, archive stats | Broad discovery; start here unless the user asked for an exact node-specific workflow |
| PDS4 | [`pds4/`](./pds4/) | PDS4-format search across bundles, collections, products, instruments, instrument hosts, investigations, targets | Broad discovery for modern PDS4 archival data |
| ODE (GEO) | [`ode-geo/`](./ode-geo/) | Orbital Data Explorer: planetary products (Mars-focused), feature classes, instruments | Narrowing on GEO-node queries (Mars-centric) |
| OPUS (RMS) | [`opus-rms/`](./opus-rms/) | Outer Planets Unified Search: ring and moon systems outer-planets data | Narrowing on RMS-node queries (outer planets, rings, moons) |
| IMG | [`img/`](./img/) | Planetary imaging: product/facet/search/count | Narrowing on IMG-node queries |
| SBN | [`sbn/`](./sbn/) | Small Bodies Node: comets, asteroids, KBOs, by object or coordinate | Narrowing on SBN queries |

## Notes

- The agent's `allowed_tools` list is the authoritative whitelist. Any tool not on that list will not be called even if the MCP server advertises it. See each family's `index.md` for the exact tool names.
- All tools execute server-side; the agent never holds credentials for the PDS nodes directly.
- See [`../contexts/reasoning-strategy.md`](../contexts/reasoning-strategy.md) for the catalog-first routing rule that governs tool selection.
