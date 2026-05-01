# akd-core

**akd-core** is AKD's foundational Python framework. Every domain agent, tool, and guardrail in AKD is built on primitives defined here.

Repo: [NASA-IMPACT/akd-core](https://github.com/NASA-IMPACT/akd-core)
Python package: `akd`

## What akd-core provides

- **`BaseAgent`** — async-first, streaming-native base class for agents. Subclasses override schemas, system prompt, and (optionally) a streaming engine.
- **`BaseTool`** — base class for tools. Define input / output schemas, implement `_arun()`.
- **`InputSchema` / `OutputSchema`** — Pydantic v2 bases that enforce docstrings and define structured I/O for agents and tools.
- **`StreamEvent` family** — a typed set of streaming events (starting, running, streaming tokens, thinking, partial output, tool calling, tool result, human input required, human response, completed, failed, partial). See [`../../docs/streaming-and-hitl.md`](../../docs/streaming-and-hitl.md).
- **`RunContext`** — execution state carried through a run (messages, usage, human response, run id).
- **`HumanTool`** — a tool that pauses an agent to request human input, integrating HITL into the streaming event model.
- **`TextOutput`** — union companion schema for agents that sometimes return free-form text (clarifications, questions).
- **Guardrails protocol** — the `GuardrailProtocol`, `CompositeGuardrail`, and the `@guardrail` / `apply_guardrails` entry points. See [`../../guardrails/`](../../guardrails/).
- **Planner primitives** — `akd.planner.llm_planner` and the agent registry used by the Flow backend to generate and execute workflow graphs.
- **Pre-built tools** — search, scrapers, resolvers, relevancy, reranker, source validator — building blocks for domain tools.
- **Pre-built agents** — literature search, gap analysis, extraction, query reformulation, relevancy, intent classification — reference implementations and composable parts.
- **LLM-agnostic provider layer** — via LiteLLM, so models across OpenAI, Anthropic, Ollama, etc. plug in uniformly.

## Design choices

- **Async-first.** All agents and tools expose `arun()` / `astream()`. The synchronous `run()` exists as a convenience but is not the primary path.
- **Streaming-native.** Every agent yields `StreamEvent`s as it runs, so UIs can render progress live.
- **HITL as a first-class capability.** Pausing and resuming on human input is built into the base class, not bolted on.
- **Schema-first.** Both inputs and outputs are Pydantic v2 models. Docstrings are required (enforced in `InputSchema`), making the generated API self-documenting.
- **LLM-agnostic.** Built on LiteLLM so providers are interchangeable. akd-ext agents (built on OpenAI Agents SDK) layer on top; akd-core itself does not lock you into one provider.

## Sections

- [`concepts.md`](./concepts.md) — the conceptual map: `BaseAgent`, `BaseTool`, schemas, streaming, HITL, provider abstraction.
- [`build-a-custom-agent.md`](./build-a-custom-agent.md) — walkthrough with code for building a minimal agent on raw `BaseAgent`.
- [`build-a-custom-tool.md`](./build-a-custom-tool.md) — walkthrough with code for building a minimal tool on raw `BaseTool`.
- [`installation.md`](./installation.md) — install options, key dependencies.

## Related

- [`../akd-ext/`](../akd-ext/) — the extensions layer; most published agents are built on `akd-ext`'s `OpenAIBaseAgent` rather than raw `BaseAgent`.
- [`../../guardrails/`](../../guardrails/) — the guardrail providers defined on top of akd-core.
- [`../../docs/streaming-and-hitl.md`](../../docs/streaming-and-hitl.md) — the streaming and HITL model.
