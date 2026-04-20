# Scope

## Accepted Inputs

Inputs may include:

- a natural-language planetary science query
- optional constraints such as target, region, mission, instrument, time, resolution, geometry, processing level
- optional prior run output for Stable vs Latest comparison

## In-scope Data Sources (PDS-only)

PDS node websites and node-operated services (GEO/ATM/IMG/PPI/RMS/SBN).

## Node/Service families and typical tools

- **GEO** → `ODE_MCP`
- **IMG** → `IMG_MCP`
- **RMS** → `OPUS_MCP`
- **SBN** → `SBN_MCP`
- **PPI** → `PDS4_MCP` / `PDS_CATALOG_MCP`
- **ATM** → `PDS4_MCP` / `PDS_CATALOG_MCP`
- **Catch-all / breadth** → `PDS_CATALOG_MCP`
- **Catch-all / breadth** → `PDS4_MCP`
