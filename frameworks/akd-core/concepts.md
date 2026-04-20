# akd-core — Concepts

A walk through the key abstractions in `akd-core`.

## Schemas

Every AKD agent and tool declares an **input schema** and an **output schema**, both subclasses of `InputSchema` / `OutputSchema` (which are Pydantic v2 models).

- `InputSchema` enforces that the schema has a non-empty docstring.
- `OutputSchema` supports a `__response_field__` class-var that names the primary text field of the output (used by some presentation paths and union-handling code).

Schemas are the agent's contract: callers construct an `InputSchema` to invoke the agent, and receive a validated `OutputSchema` (or a union such as `OutputSchema | TextOutput`).

## `BaseAgent`

The abstract base class for all AKD agents.

- Generic in its input and output schemas: `BaseAgent[InSchema, OutSchema]`.
- Subclasses must set class-level `input_schema`, `output_schema`, and `config_schema`.
- Subclasses must override `_arun()` — the core async implementation.
- `arun()` is the public entry point; it handles validation, LLM calls, message-history management, and output resolution.
- `astream()` returns an async iterator of `StreamEvent`s.
- Supports a union output schema via `OutputRoutingMixin` — an agent can return either a structured schema or a free-form `TextOutput` (useful when the agent asks a clarification question instead of producing the structured output).

## `BaseTool`

The abstract base class for tools an agent can call.

- Generic: `BaseTool[InSchema, OutSchema]`.
- Subclasses implement `async def _arun(self, params: InSchema) -> OutSchema`.
- `as_function()` produces an async callable usable by the OpenAI Agents SDK and similar orchestrators.
- `as_tool_definition()` produces an OpenAI function-calling JSON schema.
- Composable: tools can wrap other tools (see search pipelines in akd-core for examples).

## `RunContext`

Execution state that flows through an agent's run:

- `messages` — chat-completions-style conversation history.
- `run_id` — stable identifier for the run.
- `session` / metadata — caller-supplied context.
- `usage` — accumulated `RunUsage` (tokens, requests).
- `human_response` — the human's reply when the run resumes from a HITL pause.

Subclasses of `BaseAgent` receive a `RunContext` on every invocation and mutate it as needed.

## `StreamEvent` and streaming

An `astream()` invocation yields events that describe what the agent is doing. The event family includes:

- Lifecycle: `STARTING`, `RUNNING`, `COMPLETED`, `FAILED`.
- Token-level: `STREAMING` (text deltas).
- Reasoning: `THINKING` (model's reasoning summary when enabled).
- Partial output: a structured partial JSON for union / structured-output scenarios.
- Tool interaction: `TOOL_CALLING` (a tool call started), `TOOL_RESULT` (a tool result arrived).
- HITL: `HUMAN_INPUT_REQUIRED` (agent paused, waiting for a human), `HUMAN_RESPONSE` (human supplied input, agent resumed).

The Flow backend serializes these events as Server-Sent Events to the Next.js frontend; other consumers can subscribe programmatically.

## Human-in-the-Loop (HITL)

HITL is built on:

- A **`HumanTool`** that, when called, causes the agent to pause — the tool call never returns normally.
- A `HumanInputRequired` exception / event carrying the question and the tool call id.
- A resume path that injects a `human_response` (matched by tool call id) into the conversation messages and continues the run.

From the agent author's perspective, asking the human a question looks the same as calling any other tool.

## Guardrails

Implemented as a separate subsystem on top of akd-core (`akd.guardrails.*`).

- `GuardrailProtocol` — any class implementing `check()` and `acheck()`.
- Composition operators `& | >>` produce a `CompositeGuardrail`.
- `@guardrail(...)` decorator applies guardrails to an agent class.
- `apply_guardrails(...)` wraps an agent instance.

See [`../../guardrails/`](../../guardrails/) for the provider implementations (Granite Guardian, Risk Agent) and composition patterns.

## Pre-built tools

akd-core ships a library of tools that domain agents compose:

- **Search**: SearxNG, Serper, Semantic Scholar, composite search pipelines.
- **Scrapers**: web, PDF (via pymupdf), Docling.
- **Resolvers**: DOI / arXiv / ADS / Unpaywall to canonical URL resolution.
- **Evaluation helpers**: relevancy scoring, reranking, source credibility validation.

Most domain agents reach for these rather than re-implementing; akd-ext tools extend this set for NASA-specific data.

## Planner

`akd.planner.llm_planner` and `akd.planner.registry` implement the natural-language planner used by the Flow backend. Given a user request, the planner generates a workflow graph (nodes + edges) referring to registered agent ids. The Flow backend registers each published agent at startup.

See [`../../flow/architecture.md`](../../flow/architecture.md) for how the planner is integrated into Flow.

## Where to read next

- [`build-a-custom-agent.md`](./build-a-custom-agent.md) and [`build-a-custom-tool.md`](./build-a-custom-tool.md) for concrete examples.
- [`../akd-ext/concepts.md`](../akd-ext/concepts.md) for how akd-ext layers the OpenAI Agents SDK and MCP on top of akd-core.
