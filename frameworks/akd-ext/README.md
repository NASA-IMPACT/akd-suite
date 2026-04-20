# akd-ext

**akd-ext** is the AKD extensions layer — where all the published domain agents and their tools live. It sits on top of `akd-core` and adds:

- An **OpenAI Agents SDK** integration: `OpenAIBaseAgent`, a subclass of `akd-core`'s `BaseAgent` that runs its tool-call loop through the OpenAI Agents SDK runtime.
- A **Model Context Protocol (MCP)** glue layer: `HostedMCPTool` patterns, the `@mcp_tool` decorator, and a tool converter that transforms AKD `BaseTool` instances into the OpenAI Agents SDK's `FunctionTool` type.
- A **registry** and helpers for agent construction (file attachments, output routing, tool auto-conversion).
- The concrete **published domain agents**: CMR, PDS, Code Search, Astro Search, Gap, and the Closed-Loop CM1 pipeline (Capability & Feasibility Mapper, Workflow Spec Builder, Experiment Implementation, Research Report Generator).
- Domain **tools** used by those agents (e.g., `RepositorySearchTool`, `CodeSignalsSearchTool`, `SDESearchTool`).

Repo: [NASA-IMPACT/akd-ext](https://github.com/NASA-IMPACT/akd-ext)
Python package: `akd_ext`

## When to use akd-ext

- You are building an agent for AKD Flow (published-to-Flow) — start with `OpenAIBaseAgent`.
- You want MCP-backed tools, especially hosted MCP servers.
- You want the OpenAI Agents SDK's streaming, tool-use-behavior, file-attachment, and reasoning support "for free."

For pure akd-core (no OpenAI SDK, no MCP) see [`../akd-core/`](../akd-core/).

## Design patterns baked in

- **Schema-first agent authoring.** Every agent subclass declares `input_schema`, `output_schema`, and `config_schema`. Output schemas are typically a union with `TextOutput` (so the agent can ask clarifying questions when it can't return the structured form).
- **System prompt as behavior.** Process, constraints, and output format live in the prompt — not in imperative code. The prompt is a top-level string constant in the module.
- **Tools via `HostedMCPTool`.** Most published agents wire at least one hosted MCP server via OpenAI Agents SDK's `HostedMCPTool` primitive. The server label and URL env var are part of the agent's config.
- **File attachments.** `FileAttachmentMixin` lets agents accept uploaded files as context; Flow surfaces this in the UI.
- **Tool auto-conversion.** If you pass a raw `akd.tools._base.BaseTool` into an `OpenAIBaseAgentConfig`'s `tools` list, the config's validator auto-converts it to a `FunctionTool` using `function_tool(tool_converter(...))`.

## Sections

- [`concepts.md`](./concepts.md) — the OpenAI Agents SDK integration, MCP glue, tool conversion, and file attachments.
- [`build-a-custom-agent.md`](./build-a-custom-agent.md) — walkthrough using `OpenAIBaseAgent`.
- [`build-a-custom-tool.md`](./build-a-custom-tool.md) — walkthrough using `@mcp_tool` and tool conversion.
- [`agents-index.md`](./agents-index.md) — table of published domain agents with deep links.
- [`tools-index.md`](./tools-index.md) — table of ext tools and which agents use each.
- [`installation.md`](./installation.md) — install options.

## Related

- [`../akd-core/`](../akd-core/) — the base framework `akd-ext` extends.
- [`../../agents/`](../../agents/) — the consumable knowledge artifacts for each published agent.
- [`../../docs/mcp.md`](../../docs/mcp.md) — how AKD uses MCP.
