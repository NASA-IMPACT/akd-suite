# Labs — User Guide

A walkthrough for using AKD Labs as an agent developer or evaluator.

## Accessing Labs

Open [labs.akd.odsi.io](https://labs.akd.odsi.io) in a browser and log in. You'll land in the organization / project you're a member of; if you belong to multiple, use the tenant switcher in the header.

## Primary surfaces

- **Agents** — list, create, edit, chat-test, publish.
- **Runs** — benchmark runs and their results.
- **Traces** — observability into past chat turns.
- **Settings** — per-project and per-organization configuration (members, roles, default models, etc.).

## Creating an agent

1. In **Agents**, click "New agent".
2. Fill in the form:
   - **Name** — short human-readable name.
   - **Description** — one-line description.
   - **System prompt** — the agent's behavioral definition (this is the most important field).
   - **Model** — pick a model id (e.g., `gpt-5.2`, `gpt-4o`).
   - **Tools config** — JSON describing tools the agent can call (hosted MCP servers, web search, AKD tools, etc.).
   - **Model settings** — JSON with reasoning effort, store flag, truncation strategy, and other provider-specific settings.
   - **Visibility scope** — `project` (only the project's members) or `organization` (all org members).
3. Save. The agent now appears in the Agents list.

## Chat-testing an agent

Open the agent, click into the **Chat lab**, and start a conversation. Messages go through the agent's current config — every turn captures a full trace log behind the scenes. Iterate:

- Adjust the system prompt and save; the next turn uses the new prompt.
- Swap tools in the tools config.
- Change the model.
- Watch the trace (token counts, tool calls, reasoning) to understand why the agent did what it did.

## Reading a trace

Every chat turn creates a trace with:

- System prompt as sent.
- User message.
- Each tool call (name, input, output).
- Model reasoning summary (if enabled for reasoning models).
- Final response.
- Token counts (input, output, cached input, reasoning).
- Estimated cost.

Use traces to debug problems ("why did the agent skip tool X?"), tune prompts, and compare agents.

## Running benchmarks

1. Create or select a benchmark suite (a set of test prompts with expected behaviors).
2. Open **Runs** and create a new run pointed at an agent and a suite.
3. Start the run — Labs schedules it on background workers.
4. Watch progress via SSE (the UI streams status as results complete).
5. Review results: grading, cost, latency, trace links.

See [`benchmarking.md`](./benchmarking.md) for more on run grading and comparison.

## Publishing (within Labs)

Publishing inside Labs is a visibility toggle: move the agent from `project` to `organization` scope so all org members can see and use it. This is **not** the same as publishing to Flow — see [`publishing-to-flow.md`](./publishing-to-flow.md) for that story.

## Sharing results

Exports: from **Runs → Export**, download CSV/JSON/HTML of a run's results for external sharing or archival.

## Where to read next

- [`agent-development-lifecycle.md`](./agent-development-lifecycle.md) — the create-iterate-publish loop in more detail.
- [`architecture.md`](./architecture.md) — the data model underneath.
