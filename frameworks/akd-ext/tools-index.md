# akd-ext — Tools Index

Tools provided or wired by akd-ext, and which published agents use each one.

## In-process tools (Python classes in `akd_ext.tools`)

| Tool class | Purpose | Used by |
| --- | --- | --- |
| `RepositorySearchTool` | GitHub-metadata-enriched scientific code repository search | wrapped behind `repository_search_tool` on the Code Search MCP |
| `CodeSignalsSearchTool` | Static code-signal inspection (keywords, frameworks, patterns) | wrapped behind `code_signals_search_tool` on the Code Search MCP |
| `SDESearchTool` | Science Discovery Engine text search | wrapped behind `sde_search_tool` on the Code Search MCP |
| `DummyTool` | Echo tool for testing / examples | — |

These are primarily wrapped by hosted MCP servers in the published agents' configs; users typically interact with them via MCP rather than importing the Python class.

## Hosted MCP servers wired by published agents

| MCP server (label) | URL env var | Used by | What it provides |
| --- | --- | --- | --- |
| `CMR_MCP_Server` | `CMR_MCP_URL` | [CMR](../../agents/cmr/) | `search_collections`, `get_granules`, `get_collection_metadata` |
| `pds_mcp_server` | `PDS_MCP_URL` | [PDS](../../agents/pds/) | 30 allowed tools across PDS catalog, PDS4, ODE (GEO), OPUS (RMS), IMG, SBN families |
| `CMR_MCP_Server` (code-search context) | `CODE_SEARCH_MCP_URL` | [Code Search](../../agents/code-search/) | `sde_search_tool`, `repository_search_tool`, `code_signals_search_tool` |
| `ADS_Search` | `ADS_SEARCH_MCP_URL` | [Code Search](../../agents/code-search/) | `ads_search_tool`, `ads_links_resolver_tool` |
| `Astroquery_MCP_Server` | `ASTRO_MCP_URL` | [Astro Search](../../agents/astro-search/) | Astroquery execution surface + compact ADS helpers |
| `Job_Management_Server` | `EXPERIMENT_STATUS_MCP_URL` | [Experiment Implementation](../../agents/closed-loop-cm1/experiment-implementation/), [Research Report Generator](../../agents/closed-loop-cm1/research-report-generator/) | `job_submit` / `job_status` / `job_plot` |

## Built-in OpenAI Agents SDK tools used by published agents

| Tool | Used by | Purpose |
| --- | --- | --- |
| `WebSearchTool` | [Code Search](../../agents/code-search/) | Step 6 supplementary web search (completeness check against the Expected Codes checklist) |

## Tool-related infrastructure in akd-ext

- `akd_ext.mcp.converter.tool_converter` — converts an AKD `BaseTool` into a callable compatible with the OpenAI Agents SDK's `FunctionTool`.
- `akd_ext.mcp.decorators.mcp_tool` — decorator registering a `BaseTool` subclass with the MCP tool registry.
- `akd_ext.mcp.registry.MCPToolRegistry` — registry of `@mcp_tool`-decorated tools.
- `akd_ext.mcp.server` — MCP server wiring for in-process tools that should be exposed over HTTP.

## Related

- [`concepts.md`](./concepts.md) — how the tool layer fits into the OpenAI Agents SDK integration.
- [`build-a-custom-tool.md`](./build-a-custom-tool.md) — how to write a new tool.
- [`../../docs/mcp.md`](../../docs/mcp.md) — the MCP overview.
