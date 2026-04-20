# Getting started with AKD

AKD has several reader paths. Pick the one that matches your intent.

## I want to use AKD to find datasets, code, or literature

Go to **Flow** at [flow.akd.odsi.io](https://flow.akd.odsi.io). You'll land on a planner interface where you can describe your research question in natural language, and Flow will propose a multi-agent workflow (CMR for Earth-science data, Astro for astronomy data, Code Search for code, etc.), let you tune it, and then run it with streaming results you can inspect and share.

- Read [`flow/user-guide.md`](../flow/user-guide.md) for a walkthrough.
- Read [`agents/`](../agents/) for what each agent does and what data source it queries.

## I want to build a new agent

Start with **akd-ext** — this is where the domain agents live.

- Read [`frameworks/akd-ext/concepts.md`](../frameworks/akd-ext/concepts.md) to understand the `OpenAIBaseAgent` pattern, tools, and MCP integration.
- Walk through [`frameworks/akd-ext/build-a-custom-agent.md`](../frameworks/akd-ext/build-a-custom-agent.md) end-to-end; it shows a minimal custom agent.
- Use an existing agent as a reference — [`agents/cmr/`](../agents/cmr/) and [`agents/code-search/`](../agents/code-search/) are good starting points because their knowledge layer maps directly to their akd-ext source.

## I want to build a new tool

Tools are how an agent reaches external data or computation.

- Read [`frameworks/akd-core/build-a-custom-tool.md`](../frameworks/akd-core/build-a-custom-tool.md) for the base `BaseTool` pattern (pure Python).
- Read [`frameworks/akd-ext/build-a-custom-tool.md`](../frameworks/akd-ext/build-a-custom-tool.md) for how to wrap a tool as an MCP tool and make it callable by OpenAI Agents SDK agents.

## I want to experiment with agents in a playground

Go to **Labs** at [labs.akd.odsi.io](https://labs.akd.odsi.io). You can author an agent's system prompt and tool config in the web UI, chat-test it, and read the full trace of each conversation (tool calls, reasoning, token counts, cost).

- Read [`labs/user-guide.md`](../labs/user-guide.md) for the UI.
- Read [`labs/agent-development-lifecycle.md`](../labs/agent-development-lifecycle.md) for the create-test-iterate-publish loop.
- Read [`labs/publishing-to-flow.md`](../labs/publishing-to-flow.md) for how a validated Labs agent graduates to Flow.

## I want to understand AKD's safety and risk handling

- Read [`guardrails/README.md`](../guardrails/README.md) for the architecture (pluggable protocol, composition operators).
- Read [`guardrails/granite/`](../guardrails/granite/) for content moderation.
- Read [`guardrails/risk-agent/`](../guardrails/risk-agent/) for the science-focused LLM-judge.

## I want to deploy or operate AKD

- Read [`flow/deployment.md`](../flow/deployment.md) for the Flow stack (Docker, CDK, Helm).
- Read [`flow/backend.md`](../flow/backend.md) for the service dependencies (Postgres, Redis, SearxNG, Ollama, FactReasoner).

## I want to cite AKD or reference it externally

When referring to AKD, use the program name **Accelerated Knowledge Discovery** (NASA-IMPACT). Point to this wiki, or directly to the upstream repos listed in [`ecosystem.md`](./ecosystem.md).
