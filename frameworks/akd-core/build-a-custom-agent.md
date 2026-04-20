# Build a custom agent (akd-core)

This walkthrough shows how to build a minimal agent on raw `BaseAgent` from `akd-core`. For the more common AKD pattern — building on top of `OpenAIBaseAgent` with MCP tools — see [`../akd-ext/build-a-custom-agent.md`](../akd-ext/build-a-custom-agent.md).

## Prerequisites

- Python 3.12
- `akd` installed (see [`installation.md`](./installation.md))
- An LLM provider configured (OpenAI, Anthropic, Ollama, etc., via LiteLLM)

## The pattern

Every AKD agent follows the same shape:

1. Define input and output schemas.
2. Write a system prompt.
3. Create a config class.
4. Implement the agent class.
5. Instantiate and run (or stream).

## 1. Define schemas

```python
from akd._base import InputSchema, OutputSchema
from pydantic import Field


class GreetingAgentInputSchema(InputSchema):
    """Input for a greeting agent."""

    name: str = Field(..., description="The person to greet.")
    language: str = Field(default="en", description="ISO language code.")


class GreetingAgentOutputSchema(OutputSchema):
    """Output from the greeting agent."""

    __response_field__ = "greeting"
    greeting: str = Field(..., description="The generated greeting text.")
```

Docstrings are required on `InputSchema` subclasses — the base class enforces this.

## 2. Write a system prompt

```python
GREETING_SYSTEM_PROMPT = """
ROLE
You are a friendly greeting generator.

OBJECTIVE
Given a person's name and a language code, produce a short, warm greeting
in that language. Keep it to one sentence.

CONSTRAINTS
- Do not invent people, facts, or context.
- If the language is unknown or not one you can greet in, default to English.

OUTPUT FORMAT
Return the greeting in the `greeting` field.
""".strip()
```

The prompt is plain text — there is no template engine. akd-core passes it as the system message at run time.

## 3. Create a config class

```python
from akd.agents._base import BaseAgentConfig


class GreetingAgentConfig(BaseAgentConfig):
    """Config for the greeting agent."""

    system_prompt: str = Field(default=GREETING_SYSTEM_PROMPT)
    model_name: str = Field(default="gpt-4o-mini")
```

`BaseAgentConfig` already carries fields for temperature, max_tokens, debug, tools, etc. Override what you need.

## 4. Implement the agent

```python
from akd._base import TextOutput
from akd.agents._base import BaseAgent


class GreetingAgent(BaseAgent[GreetingAgentInputSchema, GreetingAgentOutputSchema]):
    """A minimal greeting agent."""

    input_schema = GreetingAgentInputSchema
    output_schema = GreetingAgentOutputSchema | TextOutput
    config_schema = GreetingAgentConfig
```

Using a **union output schema** (`GreetingAgentOutputSchema | TextOutput`) lets the agent fall back to free-form text when it cannot produce a structured response (e.g., refuses a request). `BaseAgent`'s routing mixin handles the union.

For a simple agent with no tools, nothing else is required — `BaseAgent`'s default `_arun()` handles the LLM call and output resolution.

## 5. Run it

```python
import asyncio


async def main():
    agent = GreetingAgent(GreetingAgentConfig(debug=True))
    params = GreetingAgentInputSchema(name="Nish", language="en")

    # Non-streaming
    result = await agent.arun(params)
    print(result.greeting)

    # Streaming
    async for event in agent.astream(params):
        print(event.event_type, event.message)


asyncio.run(main())
```

## Adding tools

To give the agent tools, pass tool instances through `config.tools`:

```python
from akd.tools.search import SearxNGSearchTool


class ResearchAgentConfig(BaseAgentConfig):
    system_prompt: str = Field(default=...)
    model_name: str = Field(default="gpt-4o")
    tools: list[Any] = Field(default_factory=lambda: [SearxNGSearchTool()])
```

Tool calls surface as `TOOL_CALLING` / `TOOL_RESULT` stream events during `astream()`. See [`build-a-custom-tool.md`](./build-a-custom-tool.md) for how to author new tools.

## Adding human-in-the-loop

Import `HumanTool` from `akd.tools.human` and include it in `config.tools`. When the agent calls it during a run, `astream()` will emit a `HUMAN_INPUT_REQUIRED` event and pause. Resume by passing the human's answer back through `RunContext.human_response`.

See [`../../docs/streaming-and-hitl.md`](../../docs/streaming-and-hitl.md) for the event flow.

## What to do next

- Look at a real published agent in [`agents/`](../../agents/) for the full end-to-end pattern (prompt, schemas, tools, HITL, guardrails).
- For the more common AKD pattern (OpenAI Agents SDK + MCP), follow [`../akd-ext/build-a-custom-agent.md`](../akd-ext/build-a-custom-agent.md).
- See [`concepts.md`](./concepts.md) for a deeper tour of `BaseAgent`'s internals.
