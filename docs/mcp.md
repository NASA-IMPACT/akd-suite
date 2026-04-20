# Model Context Protocol (MCP) in AKD

MCP — the **Model Context Protocol** — is the standard AKD uses to let LLM agents call tools that live outside the agent process. Most AKD data-discovery agents depend on MCP servers to reach their authoritative data sources: CMR, PDS, ADS, SDE, SIMBAD, Astroquery, and more.

## Why MCP matters for AKD

AKD agents do not embed every upstream API client. Instead, external capabilities are exposed as MCP tools — server-side processes that advertise a set of named tools with typed inputs and outputs. An AKD agent knows only:

- the MCP server's URL (usually read from an environment variable),
- a server label,
- the list of allowed tool names it may call.

The agent does not need the underlying API keys or HTTP clients. That's the MCP server's responsibility.

## How AKD uses MCP

There are two modes of MCP usage in AKD.

### Hosted MCP servers (primary pattern)

Most published AKD agents use **hosted** MCP servers — external services that expose tools over HTTP. The agent configuration wires them in via the OpenAI Agents SDK's `HostedMCPTool` primitive, which takes:

- `server_label` (human-readable name)
- `server_url` (typically populated from an env var like `CMR_MCP_URL` or `PDS_MCP_URL`)
- `allowed_tools` (the whitelist of tool names the agent is permitted to invoke)
- `require_approval` (gating behavior for tool calls)

Hosted MCP servers currently referenced by published agents include:

| MCP Server (label) | Used by agent | URL env var |
| --- | --- | --- |
| `CMR_MCP_Server` | CMR CARE Agent | `CMR_MCP_URL` |
| `pds_mcp_server` | PDS Search Agent | `PDS_MCP_URL` |
| `CMR_MCP_Server` (Code Search) | Code Search CARE Agent | `CODE_SEARCH_MCP_URL` |
| `ADS_Search` | Code Search CARE Agent | `ADS_SEARCH_MCP_URL` |
| `Astroquery_MCP_Server` | Astro Search Agent | `ASTRO_MCP_URL` |
| `Job_Management_Server` | Experiment Implementation + Research Report Generator | `EXPERIMENT_STATUS_MCP_URL` |

See each agent's [`artifacts/tools/`](../agents/) directory for per-tool MCP callouts.

### Local/in-process MCP (via `@mcp_tool`)

`akd-ext` also provides a decorator for turning an in-process `BaseTool` subclass into an MCP-compatible tool. This is useful when an agent wants to expose its own Python-defined tool to the OpenAI Agents SDK runtime without hosting a separate server. See [`frameworks/akd-ext/build-a-custom-tool.md`](../frameworks/akd-ext/build-a-custom-tool.md) for the pattern.

## What MCP gives AKD

- **Separation of concerns.** The agent stays code-light: it handles reasoning and schema. The MCP server handles upstream API specifics, auth, rate limiting, and response shaping.
- **Shared tools across agents.** The same CMR MCP server can be wired into multiple agents that need Earth science data; only the `allowed_tools` list differs.
- **Pluggability.** Swapping an upstream data source means changing one env var. No agent code change.
- **Server-side tool execution.** With `HostedMCPTool`, tool calls happen outside the agent process, so streaming events surface tool call start and tool result as first-class events the UI can render.

## What MCP does NOT mean here

- MCP servers in AKD are **not** a plugin registry. There is no runtime discovery: each agent's config statically names the servers and tool whitelist it will use.
- The Flow backend does not load MCP servers dynamically on the client side — it trusts each agent's pre-configured set.
- MCP does not mediate authorization between agents. Guardrails live separately; see [`guardrails/`](../guardrails/).

## Further reading

- [`streaming-and-hitl.md`](./streaming-and-hitl.md) — how MCP tool-call events appear in the stream.
- [`frameworks/akd-ext/concepts.md`](../frameworks/akd-ext/concepts.md) — how the OpenAI Agents SDK + HostedMCPTool fit together.
