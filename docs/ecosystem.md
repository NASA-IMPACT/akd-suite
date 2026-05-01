# The AKD ecosystem

AKD is not a single codebase. It's a layered set of repositories and products that together form the Accelerated Knowledge Discovery program. This page is the map.

## The layers

```
┌───────────────────────────────────────────────────────────────┐
│ PRODUCT SURFACE                                               │
│                                                               │
│   Flow (flow.akd.odsi.io)          Labs (labs.akd.odsi.io)    │
│   planner + workflow canvas        agent playground +         │
│   for published agents             benchmark runner           │
└───────────────┬──────────────────────────────┬────────────────┘
                │                              │
                ▼                              ▼
┌───────────────────────────────┐  ┌──────────────────────────────┐
│ RUNTIMES                      │  │ RUNTIMES                     │
│                               │  │                              │
│ akd-services                  │  │ akd-labs                     │
│ FastAPI + LangGraph           │  │ FastAPI + Next.js            │
│ akd-flow                      │  │ (multi-tenant)               │
│ Next.js 15 + React Flow       │  │                              │
└───────────────┬───────────────┘  └──────────────────────────────┘
                │
                ▼
┌────────────────────────────────────────────────────────────────┐
│ FRAMEWORKS                                                     │
│                                                                │
│ akd-ext (akd_ext)                                              │
│ domain agents (CMR, PDS, Code Search, Astro, Gap, CM1),        │
│ tools (RepositorySearch, SDE, etc.), MCP glue                  │
│                                                                │
│ akd-core (akd)                                                 │
│ base primitives: BaseAgent, BaseTool, StreamEvent, HumanTool,  │
│ guardrails (Granite, Risk Agent), planner                      │
└────────────────────────────────────────────────────────────────┘
```

## The five working repos

| Repo | Role | Language | Deployed as |
| --- | --- | --- | --- |
| [`NASA-IMPACT/akd-core`](https://github.com/NASA-IMPACT/akd-core) | Base framework — the primitives every AKD agent and tool is built on | Python 3.12 | — |
| [`NASA-IMPACT/akd-ext`](https://github.com/NASA-IMPACT/akd-ext) | Concrete domain agents + tools + MCP glue, built on top of akd-core | Python 3.12 | — |
| [`NASA-IMPACT/akd-services`](https://github.com/NASA-IMPACT/akd-services) | FastAPI backend orchestrating multi-agent workflows via LangGraph, serving the Flow API | Python 3.12 | Flow backend |
| [`NASA-IMPACT/akd-flow`](https://github.com/NASA-IMPACT/akd-flow) | Next.js 15 frontend: natural-language planner + visual workflow canvas + streaming results | TypeScript / React 19 | Flow frontend |
| [`NASA-IMPACT/akd-labs`](https://github.com/NASA-IMPACT/akd-labs) | Multi-tenant agent lab with chat-based testing, trace logs, and benchmark runner | Python + TypeScript | Labs |

Backend + frontend together are commonly called **"AKD Flow"**; the lab is called **"AKD Labs"**.

## How pieces fit in practice

### 1. A domain agent lives in akd-ext

For example, `CMRCareAgent` is a Python class in `akd-ext` that defines:
- Its reasoning strategy (via a system prompt)
- Its input and output schemas
- A tool configuration that wires in the CMR MCP server

It inherits base behavior from `akd-core` (streaming, HITL, tool-call orchestration).

See [`agents/cmr/`](../agents/cmr/) for the knowledge layer and [`frameworks/akd-ext/`](../frameworks/akd-ext/) for how such an agent is built.

### 2. The Flow backend serves the agent over HTTP

`akd-services` pins specific versions of `akd-core` and `akd-ext` as dependencies. On startup, it imports those agent classes and registers each one under a short agent id. When a user's workflow arrives, the backend looks up the agent, instantiates it, wraps it with optional guardrails, and streams its results back as Server-Sent Events.

See [`flow/architecture.md`](../flow/architecture.md).

### 3. The Flow frontend renders the workflow

`akd-flow` consumes the SSE stream and renders the running workflow visually — nodes lighting up as agents execute, results unfolding in place, and a chat-style planner for building or refining workflows in natural language.

See [`flow/frontend.md`](../flow/frontend.md).

### 4. Labs is where new agents incubate

`akd-labs` is a separate multi-tenant web app where an agent developer can:
- Author an agent's system prompt and tool config in a web UI
- Chat-test the agent interactively
- Read full traces (tool calls, reasoning, token usage, cost)
- Iterate
- Publish the agent to a project or organization scope

See [`labs/`](../labs/) for the full story, and [`labs/publishing-to-flow.md`](../labs/publishing-to-flow.md) for the path from a Labs-validated agent to a Flow-published one (today's static mechanism and the planned artifact-based future).

### 5. `akd-suite` is the canonical documentation hub

When someone says "AKD," they can point here. Every domain agent has a documented knowledge artifact under [`agents/`](../agents/). Every reusable guardrail has a page under [`guardrails/`](../guardrails/). Every framework concept has a guide under [`frameworks/`](../frameworks/). Every product has a user-facing write-up under [`flow/`](../flow/) or [`labs/`](../labs/).

## Where to read next

- [`what-is-akd.md`](./what-is-akd.md) — the program description for an outside reader.
- [`glossary.md`](./glossary.md) — acronyms and terms used throughout.
- [`mcp.md`](./mcp.md) — how AKD integrates Model Context Protocol.
- [`streaming-and-hitl.md`](./streaming-and-hitl.md) — streaming events and human-in-the-loop model.
