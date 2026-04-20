# Astro Search Agent

The **Astro Search Agent** is an AKD agent for discovering astronomical datasets across NASA astrophysics archives. It understands object names and coordinates, routes to the right archive (MAST, HEASARC, IRSA, and others), and supports literature-, coordinate-, archive-, and event-driven search patterns.

## At a glance

| | |
| --- | --- |
| **Purpose** | Help astronomy and astrophysics researchers find relevant datasets in NASA astrophysics archives via object / coordinate / literature / event-driven workflows |
| **Users** | Science researchers, PhD students, astronomers, postdoctoral researchers, and students from beginner to expert |
| **Data sources** | SIMBAD, NASA ADS, MAST, HEASARC, IRSA, NED, Gaia, VizieR, and VO services — all via the Astroquery MCP server |
| **Division** | NASA SMD — Astrophysics |
| **Reasoning patterns** | Literature-driven, coordinate-driven, archive-driven, event/alert-driven |
| **Output** | Conversational responses with summarized search context, candidate dataset listings, caveats, and offered next steps |
| **Source** | [NASA-IMPACT/akd-ext — `akd_ext/agents/astro_search_care.py`](https://github.com/NASA-IMPACT/akd-ext) |

## What a successful run looks like

The user asks about astronomical data (by object, coordinates, paper, or event). The agent:

1. Parses the request and identifies which of the four search patterns applies.
2. Checks for missing required information and asks focused clarifying questions (never guesses observation times, exposure requirements, calibration levels, or proprietary dates).
3. Resolves any object names via SIMBAD — if SIMBAD returns multiple matches, it presents options for user choice.
4. Routes to the right archive via Astroquery (HEASARC for high-energy, MAST for UV/optical space, IRSA for infrared, NED for extragalactic, Gaia for astrometry, ADS for literature).
5. Returns candidate datasets with archive, mission/instrument, observation ID, time range, exposure, processing level, access URL, and caveats.
6. Offers iteration — expanding scope, relaxing constraints, or trying additional archives, always with user approval.

## What it never does

- Executes download scripts or provides automated retrieval code
- Targets non-NASA archives (ESO, ESA-primary, CDS) without explicit user request
- Guesses critical metadata (observation times, exposures, calibration levels)
- Claims a dataset is "best" or scientifically optimal
- Fabricates observation IDs, URLs, or endpoints
- Provides access to restricted / proprietary data before release

## Deeper reading

- [`artifacts/`](./artifacts/) — the agent's full knowledge layer including its four search patterns, search workflow, special-case handling, and constraints.
- Upstream implementation: [`akd_ext/agents/astro_search_care.py`](https://github.com/NASA-IMPACT/akd-ext).
- Related: [`agents/cmr/`](../cmr/) and [`agents/pds/`](../pds/) for the Earth and Planetary counterparts; [`docs/mcp.md`](../../docs/mcp.md) for the MCP model this agent uses.
