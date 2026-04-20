# akd-ext — Concepts

A conceptual tour of what akd-ext adds on top of akd-core.

## `OpenAIBaseAgent`

A subclass of `akd-core`'s `BaseAgent` that:

- Uses the **OpenAI Agents SDK** under the hood for tool execution, streaming, and reasoning.
- Maintains `akd-core`-compatible streaming events (so the rest of AKD — Flow UI, trace logs, guardrails — sees a uniform event stream regardless of the provider).
- Supports **union output schemas** in two modes:
  - `unified_schema` (default): wrap the union in an envelope so the OpenAI SDK's structured-output support can validate against a single schema, then unwrap.
  - `multi_tool`: expose each branch of the union as a distinct function tool (`FunctionTool`); the agent picks the one that matches.
- Supports **HITL resume** by translating the akd-core `human_response` injection into OpenAI-SDK-compatible message history before continuing.
- Mixes in **`FileAttachmentMixin`** so agents can accept uploaded files as inline context (resolved and injected before the LLM call).

## `HostedMCPTool` wiring

Most domain agents wire their external capabilities through **hosted MCP servers** via `HostedMCPTool` (OpenAI Agents SDK primitive). A typical pattern in an agent's config:

```python
from agents import HostedMCPTool

def get_default_tools():
    return [
        HostedMCPTool(
            tool_config={
                "type": "mcp",
                "server_label": "CMR_MCP_Server",
                "allowed_tools": ["search_collections", "get_granules"],
                "require_approval": "never",
                "server_url": os.environ.get("CMR_MCP_URL", "<default-url>"),
            }
        )
    ]
```

This is how agents like CMR, PDS, Astro, and Code Search talk to their upstream data sources. The agent doesn't embed HTTP clients; the MCP server handles auth, rate limiting, and response shaping.

See [`../../docs/mcp.md`](../../docs/mcp.md) for the end-to-end MCP story.

## The `@mcp_tool` decorator

For Python-defined tools you want the OpenAI Agents SDK to call (without hosting a separate MCP server), apply `@mcp_tool` from `akd_ext.mcp.decorators` on your `BaseTool` subclass. The decorator registers the tool with the MCP tool registry and makes it auto-convertible via `tool_converter`.

See [`build-a-custom-tool.md`](./build-a-custom-tool.md) for the full pattern.

## Tool converter

`akd_ext.mcp.converter.tool_converter` takes an `AKDTool` (i.e., a `BaseTool` subclass) and returns an async callable with a signature the OpenAI Agents SDK can turn into a `FunctionTool`. `OpenAIBaseAgentConfig`'s tools validator invokes this automatically:

- If a tool in `config.tools` is an OpenAI SDK tool type (e.g., `HostedMCPTool`, `WebSearchTool`, `FunctionTool`), it passes through.
- If it's an `AKDTool`, it's converted to a `FunctionTool` via `function_tool(tool_converter(t))`.

So authors can mix-and-match OpenAI SDK tools and AKD tools in the same `tools` list.

## Model settings and reasoning

`OpenAIBaseAgentConfig` builds an OpenAI SDK `ModelSettings` from config values. Reasoning-capable models (o1, o3, gpt-5.x) don't accept sampling parameters like `temperature` or `top_p` — the config handles this automatically by excluding those when `reasoning_effort` is set.

## Tracing

`OpenAIBaseAgentConfig.run_config` includes a `RunConfig` with `trace_metadata` from the config's `tracing_params`. The Flow backend and Labs both exploit this for trace visibility.

## Output routing

Union outputs pass through `OutputRoutingMixin` (from akd-core). For `multi_tool` mode, output schemas become "output tools" — the agent calls the right tool to signal which branch of the union it's producing. The `_output_tool_behavior` callback validates that the resolved output passes `check_output()` before marking the run as final.

## File attachment handling

`FileAttachmentMixin.resolve_and_inject_files(run_context)` runs before each LLM call. Callers populate a file-attachment structure in the run context; the mixin converts that into inline content (base64 or URL) for the OpenAI SDK.

## The registry

`akd-core` exposes `get_agent_registry()`; the Flow backend calls `registry.register_agent(agent_class=...)` for each published agent at startup. From that point, an agent is addressable by a short string id (e.g., `cmr_care`, `code_search_care`) in planner-generated workflows.

## Where to read next

- [`build-a-custom-agent.md`](./build-a-custom-agent.md) — a worked example.
- [`build-a-custom-tool.md`](./build-a-custom-tool.md) — how to write tools that play with akd-ext.
- [`agents-index.md`](./agents-index.md) — the live roster of published agents.
- [`tools-index.md`](./tools-index.md) — the domain tools they use.
