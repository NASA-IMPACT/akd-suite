# Build a custom agent (akd-ext)

This walkthrough shows how to build a published-style AKD agent on top of `OpenAIBaseAgent`. Follow this pattern when your agent needs MCP tools, the OpenAI Agents SDK's reasoning, streaming, or file-attachment support.

## Prerequisites

- Python 3.12
- `akd` and `akd-ext` installed (see [`installation.md`](./installation.md))
- An OpenAI-compatible API key (or configured provider via LiteLLM)

## The pattern

1. Define input and output schemas.
2. Write a system prompt.
3. Create a tool config (hosted MCP, web search, AKD tools — mix as needed).
4. Create an agent config extending `OpenAIBaseAgentConfig`.
5. Implement the agent class extending `OpenAIBaseAgent`.

## 1. Define schemas

```python
from akd._base import InputSchema, OutputSchema
from pydantic import Field


class MyAgentInputSchema(InputSchema):
    """Input for my agent."""

    query: str = Field(..., description="The user's natural-language request.")


class MyAgentOutputSchema(OutputSchema):
    """Output for my agent."""

    __response_field__ = "result"
    result: str = Field(default="", description="The agent's structured answer.")
```

## 2. Write a system prompt

```python
MY_AGENT_SYSTEM_PROMPT = """
ROLE
You are a ... agent.

OBJECTIVE
...

CONSTRAINTS & STYLE RULES
- ...

PROCESS
1. ...
2. ...

OUTPUT FORMAT
...
""".strip()
```

Keep the process and constraints explicit — the akd-ext CARE-style prompts are good reference for how concrete these should be. See agents under [`../../agents/`](../../agents/) for examples.

## 3. Tool config

Decide which tools the agent needs. Most published agents wire a hosted MCP server; some also use the built-in `WebSearchTool`.

```python
import os
from agents import HostedMCPTool, WebSearchTool


def get_default_tools():
    return [
        HostedMCPTool(
            tool_config={
                "type": "mcp",
                "server_label": "my_mcp_server",
                "allowed_tools": ["my_tool_a", "my_tool_b"],
                "require_approval": "never",
                "server_url": os.environ.get("MY_MCP_URL", "<default url>"),
            }
        ),
        WebSearchTool(),
    ]
```

You can also mix in plain `BaseTool` subclasses (AKD tools); the config's tools validator auto-converts them to `FunctionTool` via the tool converter.

## 4. Agent config

```python
from typing import Any, Literal
from akd_ext.agents._base import OpenAIBaseAgentConfig


class MyAgentConfig(OpenAIBaseAgentConfig):
    """Config for my agent."""

    description: str = Field(default="My agent's one-line description (shown in Flow UI).")
    system_prompt: str = Field(default=MY_AGENT_SYSTEM_PROMPT)
    model_name: str = Field(default="gpt-5.2")
    reasoning_effort: Literal["low", "medium", "high"] | None = Field(default="medium")
    tools: list[Any] = Field(default_factory=get_default_tools)
```

`OpenAIBaseAgentConfig` already provides `model_settings`, `run_config`, and all the OpenAI-SDK plumbing; you only need to override the fields you care about.

## 5. The agent class

```python
from akd._base import TextOutput
from akd_ext.agents._base import OpenAIBaseAgent


class MyAgent(OpenAIBaseAgent[MyAgentInputSchema, MyAgentOutputSchema]):
    """My custom agent."""

    input_schema = MyAgentInputSchema
    output_schema = MyAgentOutputSchema | TextOutput
    config_schema = MyAgentConfig

    def check_output(self, output) -> str | None:
        if isinstance(output, MyAgentOutputSchema) and not output.result.strip():
            return "Result is empty. Provide the structured answer."
        return super().check_output(output)
```

- The union `MyAgentOutputSchema | TextOutput` allows clarification / free-form responses.
- `check_output()` lets you validate the resolved output before marking the run final.

## Running it

```python
import asyncio


async def main():
    agent = MyAgent(MyAgentConfig(debug=True))
    params = MyAgentInputSchema(query="Find me ...")

    # Streaming
    async for event in agent.astream(params):
        print(event.event_type, getattr(event, "message", ""))

    # Or non-streaming
    result = await agent.arun(params)
    print(result)


asyncio.run(main())
```

## Embedding additional context

If your agent needs to append large documentation (as the CM1 stages do — see [`../../agents/closed-loop-cm1/capability-feasibility-mapper/artifacts/embedded-contexts/`](../../agents/closed-loop-cm1/capability-feasibility-mapper/artifacts/embedded-contexts/)), override `_create_agent()`:

```python
from agents import Agent

class MyAgent(OpenAIBaseAgent[...]):
    ...

    def _create_agent(self) -> Agent:
        agent = super()._create_agent()
        if self.config.extra_context:
            agent.instructions += f"\n\n---\n\n## Extra Context\n\n{self.config.extra_context}"
        return agent
```

## Registering the agent with Flow

To have the agent appear as a published Flow agent, register it in `akd-services`'s backend startup (see `akd-services/src/api/runtime/app/main.py` in the upstream repo). The registration pattern is:

```python
registry.register_agent(agent_class=MyAgent, agent_id="my_agent")
```

After deployment, the Flow planner can include the agent in workflows using `"my_agent"` as its node type, and users can invoke it directly through the single-agent chat endpoint.

## What to do next

- Study a published agent's source as a reference (CMR CARE, Code Search CARE, Astro Search — all in akd-ext).
- Add guardrails — see [`../../guardrails/`](../../guardrails/).
- Make a new tool — see [`build-a-custom-tool.md`](./build-a-custom-tool.md).
- For agents that don't need the OpenAI Agents SDK, the raw pattern is in [`../akd-core/build-a-custom-agent.md`](../akd-core/build-a-custom-agent.md).
