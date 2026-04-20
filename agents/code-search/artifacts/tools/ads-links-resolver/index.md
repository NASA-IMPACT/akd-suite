---
name: ads_links_resolver_tool
description: Resolve ADS-style links and identifiers to canonical URLs
mcp_server: ADS_Search
mcp_server_url_env: ADS_SEARCH_MCP_URL
mcp_auth_env: CODE_SEARCH_ADS_SEARCH_KEY
approval: never
affordances: [resolver, mcp]
step: 5
---

# ads_links_resolver_tool

Resolve ADS-linked identifiers (bibcodes, DOIs) to canonical code or paper URLs.

## Purpose

Companion to `ads_search_tool` in Step 5 (Astrophysics only). When an ADS paper references a code via a bibcode, DOI, or legacy link, the resolver returns the best canonical URL.

## When the agent uses it

In Step 5 after `ads_search_tool` returns. Specifically when the agent needs to convert an ADS bibcode or DOI surfaced in a paper's metadata into a resolvable, current URL for a code.

## Conceptual inputs

- ADS bibcode, DOI, or link reference to resolve.

## Conceptual outputs

A resolved URL (when available) plus provenance metadata indicating how the resolution was performed.

## Rules the agent follows

- Counts against the 5-ADS-query budget shared with `ads_search_tool`.
- Use resolver only when a paper's own metadata doesn't already expose a usable URL.
- Prefer the code-site URL over ASCL landing pages (see the URL source priority rule in [`../../contexts/reasoning-strategy.md`](../../contexts/reasoning-strategy.md#step-3--context-enrichment-via-sde)).

## Upstream

Exposed via the `ADS_Search` hosted MCP.
