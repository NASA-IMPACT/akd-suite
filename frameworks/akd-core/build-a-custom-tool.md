# Build a custom tool (akd-core)

This walkthrough shows how to build a minimal `BaseTool` from `akd-core`. For tools intended to be used by `OpenAIBaseAgent`-based agents with MCP compatibility, layer on the `@mcp_tool` decorator from akd-ext — see [`../akd-ext/build-a-custom-tool.md`](../akd-ext/build-a-custom-tool.md).

## Prerequisites

- Python 3.12
- `akd` installed (see [`installation.md`](./installation.md))

## The pattern

Every AKD tool follows the same shape:

1. Define input and output schemas.
2. Define a config class (credentials, timeouts, tuning).
3. Implement the tool class with an async `_arun()`.

## 1. Define schemas

```python
from akd._base import InputSchema, OutputSchema
from pydantic import Field


class WeatherToolInputSchema(InputSchema):
    """Input for a weather lookup tool."""

    city: str = Field(..., description="City name, e.g. 'Huntsville'.")


class WeatherToolOutputSchema(OutputSchema):
    """Output from the weather tool."""

    temperature_c: float = Field(..., description="Current temperature in Celsius.")
    conditions: str = Field(..., description="Short textual condition summary.")
```

## 2. Config

```python
from akd.tools._base import BaseToolConfig


class WeatherToolConfig(BaseToolConfig):
    """Config for the weather tool."""

    api_key: str | None = Field(default=None, description="External weather API key.")
    timeout_seconds: int = Field(default=30, description="HTTP timeout.")
```

## 3. Implement the tool

```python
from akd.tools._base import BaseTool


class WeatherTool(BaseTool[WeatherToolInputSchema, WeatherToolOutputSchema]):
    """Look up current weather for a city."""

    input_schema = WeatherToolInputSchema
    output_schema = WeatherToolOutputSchema
    config_schema = WeatherToolConfig

    async def _arun(self, params: WeatherToolInputSchema) -> WeatherToolOutputSchema:
        # ... call an HTTP weather API, using self.config.api_key and self.config.timeout_seconds ...
        # Build and return an output instance.
        return WeatherToolOutputSchema(
            temperature_c=...,
            conditions=...,
        )
```

That's the complete contract. `_arun()` is async-only; there is no sync path to implement.

## Using the tool

### Directly

```python
import asyncio

tool = WeatherTool(config=WeatherToolConfig(api_key="..."))
result = await tool.arun(WeatherToolInputSchema(city="Huntsville"))
print(result.temperature_c, result.conditions)
```

### From an agent

Include the tool instance in the agent's `config.tools`:

```python
class MyAgentConfig(BaseAgentConfig):
    ...
    tools: list[Any] = Field(
        default_factory=lambda: [WeatherTool(config=WeatherToolConfig())]
    )
```

The agent's runtime will call `as_function()` to convert the tool into a callable the LLM orchestrator can use. Tool invocations surface as `TOOL_CALLING` / `TOOL_RESULT` stream events.

### As an OpenAI function-calling tool definition

```python
tool = WeatherTool(config=WeatherToolConfig())
tool_def = tool.as_tool_definition()  # JSON schema for function calling
```

## Composability

Tools can wrap other tools. A concrete example from akd-ext is `RepositorySearchTool` wrapping the underlying SDE code-search tool plus GitHub metadata enrichment — see the source in [`akd_ext/tools/code_search/`](https://github.com/NASA-IMPACT/akd-ext).

## Streaming from a tool

If your tool wants to emit fine-grained progress while it runs, use the streaming mixin patterns. In practice, most tools are fast enough that single-shot `_arun()` is fine; long-running tools can be decomposed into multiple tools or invoke sub-tools and yield incremental results through the agent.

## What to do next

- If you want your tool to be callable through MCP (the standard pattern for akd-ext agents), add the `@mcp_tool` decorator — see [`../akd-ext/build-a-custom-tool.md`](../akd-ext/build-a-custom-tool.md).
- Browse real tool implementations in the [`akd.tools`](https://github.com/NASA-IMPACT/accelerated-discovery) tree of akd-core, and [`akd_ext.tools`](https://github.com/NASA-IMPACT/akd-ext) in akd-ext.
