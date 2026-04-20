# Code Search Agent — Tools

The Code Search CARE Agent wires two hosted MCP servers plus the OpenAI Agents SDK's built-in `WebSearchTool`. Tools are used in a strict sequence (Steps 2–6 of the reasoning strategy); no tool may be re-queried once its step is complete.

## MCP servers

| Label | URL env var | Auth env var | Purpose |
| --- | --- | --- | --- |
| `CMR_MCP_Server` (named CMR in config, but serves code-search tools) | `CODE_SEARCH_MCP_URL` | `CODE_SEARCH_MCP_KEY` | NASA-verified repository search, SDE text search, code snippet inspection |
| `ADS_Search` | `ADS_SEARCH_MCP_URL` | `CODE_SEARCH_ADS_SEARCH_KEY` | ADS literature search and link resolver (Astrophysics-only) |

The agent also uses the built-in `WebSearchTool` from the OpenAI Agents SDK for Step 6 (supplementary web search for codes on the Expected Codes checklist that earlier steps missed).

## Tools by step

| Step | Tool | Backing source |
| --- | --- | --- |
| 2 — Primary discovery | [`repository-search/`](./repository-search/) | NASA-verified repo corpus |
| 3 — Context enrichment | [`sde-search/`](./sde-search/) | Science Discovery Engine |
| 4 — Deep inspection (conditional) | [`code-signals-search/`](./code-signals-search/) | Code snippet inspection |
| 5 — Astrophysics literature | [`ads-search/`](./ads-search/) | NASA ADS |
| 5 — Astrophysics link resolution | [`ads-links-resolver/`](./ads-links-resolver/) | NASA ADS |
| 6 — Supplementary web search | [`web-search/`](./web-search/) | Web |

## Notes

- All three MCP-backed code-search tools (`sde_search_tool`, `repository_search_tool`, `code_signals_search_tool`) are served by the same `CMR_MCP_Server` label — don't let the label confuse you; it hosts code-discovery affordances, not just CMR. The Code Search agent reuses this MCP server label for historical reasons.
- Web search is subject to a hard cap of 3 queries per run (see [`../contexts/reasoning-strategy.md`](../contexts/reasoning-strategy.md)), and ADS queries are capped at 5 per run.
