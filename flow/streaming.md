# Flow — Streaming

Flow uses HTTP **Server-Sent Events (SSE)** — not WebSockets — to stream workflow execution from backend to frontend. This page explains the event model and how it maps to the underlying AKD `StreamEvent` family.

## Why SSE

- **Unidirectional.** Workflow execution is backend-driven; the frontend rarely needs to push mid-run.
- **HTTP-native.** Works over existing load balancers, CDNs, and firewalls without WebSocket upgrade handling.
- **Reconnect-friendly.** Standard SSE reconnect semantics.

## On-the-wire shape

Each SSE message is a JSON object. The common fields:

| Field | Purpose |
| --- | --- |
| `type` | Event kind — `text-start`, `text-delta`, `data`, `text-end`, etc. |
| `meta` | Contextual metadata — which workflow, node id, agent id, tool call id, step type. |
| `delta` | For `text-delta` events, the appended text chunk. |
| `output` | For `data` events, structured output (JSON) from a node's completion. |

Example sequence for a two-node workflow (`node_a → node_b`):

```
{"type": "text-start", "meta": {"event": "workflow_started", "workflow_id": "..."}}
{"type": "text-delta", "meta": {"event": "node_running", "node": "node_a"}, "delta": "Searching CMR..."}
{"type": "data",       "meta": {"event": "completed", "node": "node_a"}, "output": {...}}
{"type": "text-delta", "meta": {"event": "node_running", "node": "node_b"}, "delta": "Analyzing gaps..."}
{"type": "data",       "meta": {"event": "completed", "node": "node_b"}, "output": {...}}
{"type": "text-end",   "meta": {"event": "workflow_complete"}}
```

## Mapping to akd-core `StreamEvent`s

Internally, each agent yields typed `StreamEvent`s (see [`../docs/streaming-and-hitl.md`](../docs/streaming-and-hitl.md)). The backend translates those into SSE messages:

| akd-core event | Typical SSE `type` / `meta.event` |
| --- | --- |
| `STARTING` | `text-start` with `event: node_started` |
| `RUNNING` | `text-delta` with `event: node_running` |
| `STREAMING` (token deltas) | `text-delta` with accumulated content |
| `THINKING` (reasoning summary) | `text-delta` with `event: node_thinking` |
| `TOOL_CALLING` | `data` with `event: tool_calling` and the tool name + arguments |
| `TOOL_RESULT` | `data` with `event: tool_result` and the tool output |
| `PARTIAL` (partial structured output) | `data` with `event: node_partial_output` |
| `HUMAN_INPUT_REQUIRED` | `data` with `event: human_input_required` and the question |
| `HUMAN_RESPONSE` | `data` with `event: human_response` |
| `COMPLETED` | `data` with `event: completed` and the final `output` |
| `FAILED` | `data` with `event: failed` and the error detail |

The frontend's `lib/streaming.ts` normalizes these into a uniform `NormalizedStreamEvent` shape for UI consumption.

## Frontend handling

When the user runs a workflow:

1. `lib/api/workflows.ts::executeWorkflow()` opens a fetch with `Accept: text/event-stream`.
2. An async iterator reads chunks, splits on `\n\n` boundaries, and parses each JSON message.
3. Each normalized event is dispatched to a callback that updates the Zustand store (active node, streaming text, partial output, etc.).
4. React components re-render in real time.

## Human-in-the-loop over SSE

When an agent needs a human response:

1. Backend emits a `human_input_required` data event.
2. Frontend renders a prompt (inline form, confirmation dialog, or chat-style question).
3. The SSE stream stays open but no more progress events arrive for that node.
4. When the user submits an answer, the frontend calls a resume endpoint.
5. Backend injects the human response into the LangGraph checkpoint, resumes the run, and new events begin flowing.

## Error handling

- Network drops: the frontend can reconnect and re-fetch the workflow's latest state to recover.
- Backend errors during a node: surfaced as a `failed` event with details; the UI marks the node as errored and halts the workflow.
- Malformed events: the frontend logs and continues (best-effort parsing).

## Related

- [`../docs/streaming-and-hitl.md`](../docs/streaming-and-hitl.md) — the abstract event model.
- [`api-overview.md`](./api-overview.md) — endpoints that return SSE.
- [`frontend.md`](./frontend.md) — how the UI consumes events.
- [`backend.md`](./backend.md) — how the backend produces them.
