# Streaming and Human-in-the-Loop

AKD is streaming-native and human-in-the-loop (HITL) by default. This page explains both from a conceptual standpoint.

## Streaming events

Every AKD agent exposes an `astream()` method that yields a sequence of typed **stream events** as it runs. The UI watches these events to render progress in real time; the backend persists them as trace logs.

The core event types include:

- **Starting / Running / Completed** — lifecycle markers.
- **Streaming tokens** — partial text deltas from the LLM, useful for typewriter-style UIs.
- **Thinking (reasoning)** — reasoning-summary content from reasoning-capable models.
- **Partial output** — structured partial JSON objects while the model fills in a schema.
- **Tool calling** — a tool invocation has started; includes tool name, call id, and arguments.
- **Tool result** — a tool has returned; includes the tool call id and the tool output.
- **Human input required** — the agent has paused and is waiting for human input.
- **Human response** — a queued human response has been injected, resuming the run.
- **Failed** — the run terminated with an error.

In Flow, these events cross the wire as HTTP Server-Sent Events (SSE). The frontend parses them and updates the active workflow node, log panel, and output area accordingly.

## Human-in-the-Loop (HITL)

HITL is a first-class capability, not an afterthought. Agents can pause mid-run to ask the user a question, request approval, or confirm a next step. This is implemented as a tool call that never returns from the tool side — instead, the agent emits a **Human input required** event, the run is persisted, and execution resumes later when a user provides the human response (which the runtime re-injects as the tool result).

### When HITL appears

- **Clarification** — before searching, a CMR or PDS agent may ask for missing scope (spatial, temporal, variable).
- **Approval gates** — the CM1 pipeline pauses between stages so a researcher can approve the feasibility report before workflow-spec generation, the spec before implementation, and so on.
- **Indirect inference permission** — the CMR agent asks before doing multi-hop discovery via Semantic Scholar.
- **Scope expansion** — the Astro agent asks before widening search radius or adding another archive.

### What HITL guarantees

- A paused agent's state is persisted (in Flow, via LangGraph's PostgreSQL checkpointer). Resumes work across process restarts.
- The human response is delivered as a standard tool result, so the model sees it in the same message history it expects.
- UIs can render the pause clearly — as a form field, a confirmation prompt, or a chat bubble — without the agent code caring about the UI.

### What HITL does NOT replace

- HITL is not a substitute for **guardrails**. Guardrails automatically filter content at input or output; HITL is for decision points where the agent genuinely wants a human to choose. See [`guardrails/`](../guardrails/).

## Why these patterns matter for AKD

Scientific workflows rarely complete in one shot. The CMR-CARE loop, the PDS catalog-first routing, and the CM1 closed-loop pipeline all interleave autonomous steps with researcher decisions. Streaming makes those steps legible; HITL makes the decisions safe.

## Further reading

- [`mcp.md`](./mcp.md) — how tool calls (MCP or in-process) appear in the stream.
- [`flow/streaming.md`](../flow/streaming.md) — how Flow's UI renders stream events.
- [`frameworks/akd-core/concepts.md`](../frameworks/akd-core/concepts.md) — the underlying `StreamEvent` and `HumanTool` abstractions.
