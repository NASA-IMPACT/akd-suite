---
name: sde_search_tool
description: Science Discovery Engine text search over NASA institutional documentation
mcp_server: CMR_MCP_Server
mcp_server_url_env: CODE_SEARCH_MCP_URL
mcp_auth_env: CODE_SEARCH_MCP_KEY
approval: never
affordances: [search, mcp]
step: 3
---

# sde_search_tool

Search NASA's Science Discovery Engine (SDE) — institutional documentation, reports, technical pages, and ASCL entries — for context that enriches the candidate list.

## Purpose

Step 3 of the reasoning strategy: context enrichment. SDE gives the agent access to NASA reports and documentation that frequently reference specific software by name. It is **also a discovery channel**: SDE may surface additional repositories not found in Step 2.

## When the agent uses it

Once per run in Step 3. The agent queries SDE with the user's core scientific terms.

**Domain-specific yield:**

- **Strongest** for Earth Science, Heliophysics, and Planetary Science queries — NASA institutional documentation references domain-specific software often.
- **Lighter yield** for Astrophysics — many astrophysics community codes are documented outside NASA channels. For Astrophysics, SDE is limited to one query and ADS (Step 5) takes over as the primary enrichment channel.

## Conceptual inputs

- One search query (free text, scientific terms).
- Optional filters (document type, time range) if supported by the underlying SDE service.

## Conceptual outputs

A list of SDE documents. Each document includes:

- Title and document type
- `url` (for ASCL entries, this is the ASCL landing page — see rules below)
- `content` field with the document body text

When an SDE result is an ASCL entry, the `content` field contains the ASCL page text. The agent extracts two critical fields from that text:

1. The **"Code site" URL** (GitHub / GitLab / project website) — used as the code's primary URL.
2. The **"Described in" bibcode** — the code's canonical method paper; used as the first entry in the output's `ads_evidence.bibcodes` list.

## URL source priority (critical rule)

- URLs in old ADS papers and web sources are frequently dead or outdated.
- When an ASCL entry exists, use the code-site URL the ASCL entry links to — **never** the ASCL landing page itself (e.g., do NOT use `ascl.net/XXXX.XXX` as the code URL).
- If no ASCL entry exists, prefer (1) GitHub/GitLab repository URL, (2) official project website found via web search.
- When no "Described in" bibcode exists, use the ASCL record bibcode (e.g., `YYYYascl.soft.XXXXXZ`) as a fallback reference.

## Rules the agent follows

- Do not treat SDE as a re-validation channel for repositories already found in Step 2; use it for enrichment and discovery.
- If SDE surfaces an ASCL entry, extract code-site URL and bibcode from the content text — not from the landing-page URL.
- Record provenance as "SDE Text Search" in the output.

## Upstream

Backed by `akd_ext/tools/sde_search.py`. Exposed via the `CMR_MCP_Server` hosted MCP.
