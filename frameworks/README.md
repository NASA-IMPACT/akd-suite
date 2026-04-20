# Frameworks

The two Python packages that form AKD's foundation.

| Package | Repo | Role |
| --- | --- | --- |
| [`akd-core`](./akd-core/) | [NASA-IMPACT/accelerated-discovery](https://github.com/NASA-IMPACT/accelerated-discovery) | Base primitives: `BaseAgent`, `BaseTool`, streaming, human-in-the-loop, guardrails, planner |
| [`akd-ext`](./akd-ext/) | [NASA-IMPACT/akd-ext](https://github.com/NASA-IMPACT/akd-ext) | Domain extensions: OpenAI Agents SDK integration, MCP glue, the published domain agents and tools |

## When to use which

- **Build on `akd-core`** when you need the raw base classes (async-first agents, streaming events, HITL support) and aren't tied to the OpenAI Agents SDK.
- **Build on `akd-ext`** when you want the full opinionated stack: `OpenAIBaseAgent`, hosted MCP tool integration, automatic AKD-tool-to-OpenAI-function-tool conversion, and patterns already proven by the published agents.

Most published agents in AKD are `akd-ext` agents built on top of `OpenAIBaseAgent`. See [`akd-ext/build-a-custom-agent.md`](./akd-ext/build-a-custom-agent.md) for the canonical walkthrough.

## What's in here

Unlike the [`agents/`](../agents/) directory which is the code-free knowledge layer, the `frameworks/` docs are written for developers and may include code examples in the `build-a-custom-*` guides. Other pages (overview, concepts, installation) stay conceptual.

- [`akd-core/README.md`](./akd-core/README.md) — what the base framework provides.
- [`akd-core/concepts.md`](./akd-core/concepts.md) — schemas, async-first execution, streaming, HITL, LLM-agnostic provider model.
- [`akd-core/build-a-custom-agent.md`](./akd-core/build-a-custom-agent.md) — minimal agent using raw `BaseAgent`.
- [`akd-core/build-a-custom-tool.md`](./akd-core/build-a-custom-tool.md) — minimal tool using raw `BaseTool`.
- [`akd-core/installation.md`](./akd-core/installation.md) — install paths and dependencies.
- [`akd-ext/README.md`](./akd-ext/README.md) — what ext provides on top of core.
- [`akd-ext/concepts.md`](./akd-ext/concepts.md) — `OpenAIBaseAgent`, `HostedMCPTool`, `@mcp_tool` decorator.
- [`akd-ext/build-a-custom-agent.md`](./akd-ext/build-a-custom-agent.md) — custom agent using `OpenAIBaseAgent`.
- [`akd-ext/build-a-custom-tool.md`](./akd-ext/build-a-custom-tool.md) — custom tool wrapped via `@mcp_tool`.
- [`akd-ext/agents-index.md`](./akd-ext/agents-index.md) — table of published ext agents with deep links into `agents/`.
- [`akd-ext/tools-index.md`](./akd-ext/tools-index.md) — table of ext tools and which agents use each.
- [`akd-ext/installation.md`](./akd-ext/installation.md) — install paths and dependencies.
