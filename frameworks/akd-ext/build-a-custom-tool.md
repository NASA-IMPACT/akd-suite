# Build a custom tool (akd-ext)

Two common shapes for an akd-ext-compatible tool:

1. **In-process AKD tool** — a `BaseTool` subclass decorated with `@mcp_tool`. Auto-converted to an OpenAI Agents SDK `FunctionTool` when an `OpenAIBaseAgent` references it.
2. **Hosted MCP tool** — served by an external MCP server; referenced through `HostedMCPTool` inside an agent's config. (This walkthrough focuses on case 1; the hosted case is covered in [`../../docs/mcp.md`](../../docs/mcp.md) at a conceptual level.)

## Prerequisites

- Python 3.12
- `akd` and `akd-ext` installed (see [`installation.md`](./installation.md))

## The pattern (in-process tool)

1. Define input and output schemas.
2. Define a tool config.
3. Implement the tool class — subclass `BaseTool`, add `@mcp_tool`, implement `_arun()`.
4. Use the tool in an `OpenAIBaseAgent`'s config — it will be auto-converted.

## 1. Define schemas

```python
from akd._base import InputSchema, OutputSchema
from pydantic import Field


class LookupToolInputSchema(InputSchema):
    """Input for the lookup tool."""

    term: str = Field(..., description="The term to look up.")


class LookupToolOutputSchema(OutputSchema):
    """Output from the lookup tool."""

    definition: str = Field(..., description="Short definition.")
    source_url: str | None = Field(default=None, description="Where the definition came from.")
```

## 2. Config

```python
from akd.tools._base import BaseToolConfig


class LookupToolConfig(BaseToolConfig):
    """Config for the lookup tool."""

    timeout_seconds: int = Field(default=10)
```

## 3. Implement the tool

```python
from akd.tools._base import BaseTool
from akd_ext.mcp.decorators import mcp_tool


@mcp_tool
class LookupTool(BaseTool[LookupToolInputSchema, LookupToolOutputSchema]):
    """Look up a term in a dictionary-like source."""

    input_schema = LookupToolInputSchema
    output_schema = LookupToolOutputSchema
    config_schema = LookupToolConfig

    async def _arun(self, params: LookupToolInputSchema) -> LookupToolOutputSchema:
        # ... perform the lookup ...
        return LookupToolOutputSchema(definition=..., source_url=...)
```

That's it. The `@mcp_tool` decorator registers the tool with akd-ext's MCP tool registry.

## 4. Use in an `OpenAIBaseAgent`

Because `OpenAIBaseAgentConfig`'s tools validator auto-converts AKD tools, you can just include the instance:

```python
from akd_ext.agents._base import OpenAIBaseAgentConfig


class ResearchAgentConfig(OpenAIBaseAgentConfig):
    tools: list[Any] = Field(
        default_factory=lambda: [LookupTool(config=LookupToolConfig())]
    )
```

Under the hood, the config's validator runs `function_tool(tool_converter(LookupTool_instance))` to produce an OpenAI SDK `FunctionTool` the agent can invoke.

## Using the tool directly (without an agent)

The tool can still be used standalone — it's a normal `BaseTool`:

```python
import asyncio

tool = LookupTool(config=LookupToolConfig())
result = await tool.arun(LookupToolInputSchema(term="CARE loop"))
print(result.definition)
```

## Hosted MCP tools

If you want to expose your tool over HTTP so multiple agents (possibly in different processes) can share it, host it behind an MCP server and wire it into an agent via `HostedMCPTool` — see [`build-a-custom-agent.md`](./build-a-custom-agent.md) for that pattern and [`../../docs/mcp.md`](../../docs/mcp.md) for the conceptual overview.

Hosted MCP tools have advantages: centralized auth, rate limiting, caching, and the ability to wrap upstream APIs that aren't amenable to being embedded. Most published AKD agents use hosted MCPs for their authoritative data-source calls.

## What to do next

- Study real tool implementations in `akd_ext/tools/` (e.g., `RepositorySearchTool`, `CodeSignalsSearchTool`, `SDESearchTool`).
- For the base-only pattern (no `@mcp_tool`, no OpenAI Agents SDK), see [`../akd-core/build-a-custom-tool.md`](../akd-core/build-a-custom-tool.md).
