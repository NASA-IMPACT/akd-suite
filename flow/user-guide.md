# Flow — User Guide

A step-by-step walkthrough for a scientist or researcher using AKD Flow for the first time.

## Accessing Flow

Open [flow.akd.odsi.io](https://flow.akd.odsi.io) in a browser. You'll be asked to log in (or register). Once authenticated, you land on the main workspace.

## The workspace

The workspace has three primary regions:

- **Left sidebar** — your chat history. Each conversation represents one research goal, with its associated workflow. You can rename, share, import, or delete chats.
- **Center — Planner chat** — a chat interface where you describe what you want to do in natural language. The planner converts your request into a workflow.
- **Right — Workflow canvas** — a visual graph editor (React Flow) where the generated workflow appears. Nodes are agents; edges are the order of execution.

## Starting a new conversation

Click "New chat" in the left sidebar. Type your research question into the Planner:

> "Find recent NASA Earthdata datasets about sea-ice extent since 2010, then find any code repositories for analyzing that data, and identify research gaps."

The planner thinks for a moment and produces a proposed workflow — typically a small sequence like CMR → Code Search → Gap. You see it as:

- A text summary in the chat (`RetrievedWorkflowCard`).
- A graph on the canvas with three nodes connected in order.

## Reviewing and refining

You can iterate with the planner:

- Add more: "Also check Astro archives for related observations."
- Simplify: "Drop the Code Search step."
- Swap agents: "Use PDS instead of CMR."

Each message returns an updated workflow card and canvas.

## Tuning parameters

Click a node on the canvas to expand it. Each agent has input fields matching its input schema — for CMR, you'll see the free-text query; for Gap, you'll attach a corpus of papers; etc. Fill what's needed.

If a node has guardrails enabled, you'll see a badge on the node indicating status (passing / flagged).

## Running

Click **Run Workflow**. The backend begins executing the graph:

- The canvas highlights the currently running node.
- A progress panel shows streaming updates: the node's tool calls, token deltas, reasoning summaries.
- When a node completes, its output appears in place (markdown for prose-like outputs, JSON / tables for structured outputs).
- If a node pauses to ask a human question (HITL), a prompt appears. Answer it and the run resumes.

## Watching and inspecting

While the workflow runs:

- **Stream panel** — the live event log. Useful for debugging if a node misbehaves.
- **Guardrail badges** — show if an input or output was flagged.
- **Fact verification** — on eligible agent outputs, you can click a "Verify" affordance to run the separate FactReasoner service over the claims.

## Completion and sharing

When the workflow finishes, all outputs are saved. You can:

- Revisit the chat later — everything is persisted.
- **Share** the chat via a public URL — anyone with the link can view the workflow and outputs.
- **Import** a shared chat into your own workspace to re-run or modify it.

## Single-agent chat

Alongside workflows, Flow offers a simpler **single-agent chat** mode via the agent picker. Use this when you want to talk to one agent directly (e.g., just CMR) without composing a full workflow. The backend endpoint is `/api/v1/agent_chats/{agent_name}`; the UI surfaces it as an agent-specific chat thread.

## Files

Some agents accept uploaded files (e.g., the Gap Agent reads a paper corpus). Use the file-upload affordance on the node to attach files; the backend stores them, and the agent receives them as inline context at runtime.

## What the published agents do

See the agent directories for a reader-focused description of each:

- [`agents/cmr/`](../agents/cmr/) — Earth science / CMR data discovery
- [`agents/pds/`](../agents/pds/) — Planetary Data System
- [`agents/code-search/`](../agents/code-search/) — Scientific code repository discovery
- [`agents/astro-search/`](../agents/astro-search/) — Astrophysics archives
- [`agents/gap/`](../agents/gap/) — Research-gap detection
- [`agents/closed-loop-cm1/`](../agents/closed-loop-cm1/) — End-to-end CM1 simulation research pipeline

## Getting help

- Open the live site and click your avatar for account / support info.
- For documentation questions about an agent's behavior, start in its [`agents/<name>/artifacts/`](../agents/) directory.
- For framework questions, see [`frameworks/`](../frameworks/).
