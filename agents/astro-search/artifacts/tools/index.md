# Astro Search Agent — Tools

The Astro Search Agent wires a single hosted MCP server, `Astroquery_MCP_Server`, which exposes a generic Astroquery execution surface plus a pair of compact ADS helpers.

## MCP server

- **Label**: `Astroquery_MCP_Server`
- **URL env var**: `ASTRO_MCP_URL`
- **Approval gating**: never

## Allowed tools

The MCP server whitelists the following tools. Rather than separate directories per affordance, they are documented here as a family because they are all served through the same server and cover the same "discover + execute Astroquery" surface.

### Astroquery discovery and execution

- `astroquery_list_modules` — enumerate available Astroquery modules (MAST, HEASARC, IRSA, NED, SIMBAD, Gaia, VizieR, etc.).
- `astroquery_list_functions` — list callable functions within a given Astroquery module.
- `astroquery_get_function_info` — inspect a function's signature and documentation.
- `astroquery_check_auth` — verify authentication for modules that require it.
- `astroquery_execute` — execute a specific Astroquery function with arguments. This is the primary execution surface — it's how the agent calls `simbad_query_object`, `heasarc_query_region`, `mast_query_object`, `irsa_query_region`, `ned_query_region`, `gaia_cone_search`, `vizier_query_region`, and all the other specific Astroquery functions referenced in the agent's prompt.

### ADS (compact)

- `ads_query_compact` — lightweight ADS literature search returning minimal fields.
- `ads_get_paper` — retrieve a specific paper's details.

## How the agent reasons about tools

The agent's prompt references many Astroquery-specific function names (e.g., `simbad_query_bibobj`, `heasarc_query_region`, `mast_get_product_list`). These are **not** separate MCP tools — they are **specific Astroquery functions invoked through `astroquery_execute`**. The `astroquery_list_modules` + `astroquery_list_functions` + `astroquery_get_function_info` triad lets the agent inspect what's available before calling `astroquery_execute`.

Refer to [`../contexts/archives-and-services.md`](../contexts/archives-and-services.md) for the mapping of science cases to specific Astroquery functions, and [`../contexts/reasoning-strategy.md`](../contexts/reasoning-strategy.md) for the execution workflow.
