---
name: ode_geo
description: Orbital Data Explorer (GEO node) — Mars-focused planetary product search
mcp_server: pds_mcp_server
mcp_server_url_env: PDS_MCP_URL
mcp_auth_env: PDS_MCP_KEY
approval: never
affordances: [search, node-specific, mcp]
---

# ODE Tools (`ode_*`)

The ODE (Orbital Data Explorer) family is a node-specific tool set for the **GEO node**, primarily Mars-centric planetary products: surface products, feature-class geometry, instrument listings.

## Allowed tools

- `ode_search_products_tool` — product search at the GEO node.
- `ode_count_products_tool` — count matches for a product search.
- `ode_list_instruments_tool` — enumerate instruments indexed by ODE.
- `ode_list_feature_classes_tool` — enumerate feature classes (surface features).
- `ode_list_feature_names_tool` — enumerate named features.
- `ode_get_feature_bounds_tool` — retrieve bounding geometry for a named feature.

## When the agent uses this family

Per the reasoning strategy's routing table, **GEO → `ODE_MCP`**. The agent calls ODE after catalog-first discovery identifies the query as GEO-node-appropriate, or when the user has explicitly targeted a Mars surface feature or mission workflow. Exact-identifier queries with `ODE_ID` can route directly here.

## Conceptual inputs

- Product search: target, mission, instrument, spatial bounds, feature name
- Feature queries: feature class, feature name
- Count: same filter set as search, returns aggregate hit count

## Conceptual outputs

- Product records with `PRODUCT_ID`, access URLs, and metadata
- Feature class / feature name lists
- Feature bounds (geometry)

## Rules the agent follows

- Use as a narrowing step, not as the starting point — unless catalog-first discovery is impossible or a specific GEO workflow is required.
- Confirm PDS4 counterpart existence when feasible (ODE is traditionally PDS3-heavy, but PDS4 migrations exist for select collections).

## Upstream

Exposed via the `pds_mcp_server` hosted MCP. Backs onto the ODE web services at the GEO node.
