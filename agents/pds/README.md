# PDS Agent

The **PDS Search Agent** is an AKD agent for discovering datasets and products across NASA's Planetary Data System (PDS). It performs **catalog-first routing**: broad discovery with cross-node PDS tools, then narrowing via node-specific services when needed. It returns both PDS4 and PDS3 versions of the same data when available.

## At a glance

| | |
| --- | --- |
| **Purpose** | Translate a planetary science question into bounded searches across PDS discovery tools and return relevant bundles/collections/datasets/products with stable identifiers and download paths |
| **Users** | Planetary scientists and researchers |
| **Data source** | PDS node websites and node-operated services (GEO, ATM, IMG, PPI, RMS, SBN) |
| **Division** | NASA SMD — Planetary Science |
| **Reasoning pattern** | Catalog-first broad search → narrow with node-specific tools; cross-version (PDS3/PDS4) discovery |
| **Output** | Structured sections: clarifying questions, interpreted scope, search plan, curated candidate shortlist, candidate metadata, decision gate |
| **Source** | [NASA-IMPACT/akd-ext — `akd_ext/agents/pds_search_care.py`](https://github.com/NASA-IMPACT/akd-ext) |

## What a successful run looks like

The user asks a planetary science question. The agent:

1. Interprets the request without inventing facts; asks for clarification only when the query is too ambiguous or broad.
2. Chooses the right granularity — dataset-level or product-level.
3. Searches broad first via `PDS_CATALOG_MCP` and/or `PDS4_MCP`; checks for both PDS4 and PDS3 representations.
4. Narrows with node-specific tools (ODE/GEO for Mars, IMG for imaging, OPUS/RMS for ring-moon systems, SBN for small bodies) when needed.
5. Returns a ranked candidate shortlist with identifiers, version info, parent context, download paths, and explicit missing-metadata lists.
6. Offers a decision gate for refinement or comparison.

## What it never does

- Downloads, carts, emails, or password-protected workflows
- Scientific interpretation or conclusions
- Non-PDS result sources
- Invented identifiers or metadata
- Subjective endorsement language ("best," "top," "most suitable")
- Bulk scraping or unbounded retrieval
- Credential or access-control bypass

## Deeper reading

- [`artifacts/`](./artifacts/) — role, reasoning strategy, constraints, output format, and tool families.
- Upstream implementation: [`akd_ext/agents/pds_search_care.py`](https://github.com/NASA-IMPACT/akd-ext).
- Related: [`agents/cmr/`](../cmr/) (sister Earth-science agent), [`agents/astro-search/`](../astro-search/) (astrophysics counterpart), [`docs/mcp.md`](../../docs/mcp.md).
