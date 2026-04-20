# Flow — Architecture

AKD Flow is the pairing of a FastAPI backend ([`akd-framework`](https://github.com/NASA-IMPACT/akd-framework)) with a Next.js frontend ([`ad-nextjs-ui`](https://github.com/NASA-IMPACT/ad-nextjs-ui)). Together they host the published AKD agents and expose them to users through a natural-language planner and a visual workflow canvas.

## The big picture

```
                    ┌──────────────────────────────┐
                    │  User's browser              │
                    │                              │
                    │  ad-nextjs-ui (Next.js 15)   │
                    │  - Planner chat              │
                    │  - React Flow canvas         │
                    │  - SSE stream consumer       │
                    └────────────────┬─────────────┘
                                     │ HTTPS
                                     │ JWT auth
                                     │ SSE streaming
                                     ▼
                    ┌──────────────────────────────┐
                    │  akd-framework (FastAPI)     │
                    │                              │
                    │  - /api/v1/workflows/* (CRUD │
                    │    + execution)              │
                    │  - /api/v1/chats/* (planner) │
                    │  - /api/v1/agent_chats/*     │
                    │  - /api/v1/files/* (uploads) │
                    │  - /api/v1/auth/*            │
                    │                              │
                    │  Agents registered at        │
                    │  startup from akd-ext:       │
                    │  CMR, PDS, Code Search,      │
                    │  Astro, Gap, CM1 stages      │
                    │                              │
                    │  LangGraph workflow engine   │
                    │  Planner (akd.planner.*)     │
                    │  Guardrails wrappers         │
                    └─────┬──────────┬─────────┬───┘
                          │          │         │
                          ▼          ▼         ▼
                    ┌─────────┐ ┌────────┐ ┌──────────┐
                    │Postgres │ │ Redis  │ │ External │
                    │15       │ │  7     │ │ services │
                    │         │ │        │ │          │
                    │ Workflow│ │ Rate   │ │ SearxNG  │
                    │ storage │ │ limits │ │ Ollama   │
                    │ Chat    │ │        │ │ Fact-    │
                    │ history │ │        │ │ Reasoner │
                    │ LangGraph│ │        │ │ Hosted   │
                    │ check-  │ │        │ │ MCPs     │
                    │ points  │ │        │ │ (CMR,    │
                    └─────────┘ └────────┘ │ PDS, ADS,│
                                           │ Astro,   │
                                           │ Jobs)    │
                                           └──────────┘
```

## Backend summary (akd-framework)

- **FastAPI** service, Python 3.12, typically served via `uvicorn` on port 8001.
- **LangGraph** orchestrates multi-agent workflows as state graphs, with PostgreSQL as the checkpointer (workflows resume cleanly across process restarts — important for HITL).
- **Agents** are registered at startup from `akd-ext` (see [`../frameworks/akd-ext/agents-index.md`](../frameworks/akd-ext/agents-index.md)). Registration is static — new agents require a backend redeploy.
- **Planner** (`akd.planner.llm_planner`) generates workflow blueprints from user natural-language requests; the planner sees the registered agents as the available node types.
- **Guardrails** (see [`../guardrails/`](../guardrails/)) wrap agents per their per-agent config, applied at input / output / both.
- **Rate limiting** via FastAPI-Limiter + Redis.
- **Auth** via JWT (`fastapi-users`).
- **File attachments** (uploaded context documents) are accepted via a dedicated endpoint and resolved into agent run contexts.

See [`backend.md`](./backend.md) for a deeper tour.

## Frontend summary (ad-nextjs-ui)

- **Next.js 15** (App Router) + **React 19** + **TypeScript 5**.
- **Tailwind CSS 4** + **Radix UI** for UI primitives.
- **React Flow** (`@xyflow/react`) for the visual workflow canvas.
- **Zustand** for client-side state.
- **Streamdown** for parsing the backend's SSE event stream.
- Two primary flows: the **Planner chat** (generate / refine workflows from natural language) and the **Workflow canvas** (run, watch, tune workflows).

See [`frontend.md`](./frontend.md) for a deeper tour.

## How a single user action plays out

Using "Find climate datasets about sea-ice" as an example:

1. **User types** into the Planner chat. Frontend posts to `/api/v1/chats/` with the message.
2. **Backend planner** (via `akd.planner.llm_planner`) generates a workflow blueprint — perhaps `CMR → Code Search → Gap` — using the registered agent ids.
3. **Frontend** renders the blueprint: a `RetrievedWorkflowCard` in the chat, plus the workflow on the canvas.
4. **User tunes** parameters (spatial bounds, keywords) by editing fields on the canvas nodes.
5. **User clicks Run.** Frontend posts to `/api/v1/workflows/{id}/run`. Backend returns a `StreamingResponse` (SSE).
6. **LangGraph executes** the graph. Each node instantiates the registered agent, wraps with guardrails, and streams `StreamEvent`s back.
7. **Frontend parses** the SSE stream, highlights active nodes, renders token deltas, tool calls, and tool results in real time.
8. **On completion**, outputs are persisted to Postgres and displayed on the canvas.
9. **User can share** the chat + workflow via a public URL; others import it into their own workspaces.

See [`user-guide.md`](./user-guide.md) for a user-facing version.

## External services the Flow stack depends on

- **PostgreSQL 15** — workflows, chats, agent chats, users; also LangGraph checkpoints.
- **Redis 7** — rate-limit counters.
- **SearxNG** — meta-search engine used by tools that perform web search (supplemental to hosted MCPs).
- **Ollama** — local LLM runtime used for some guardrail providers (Granite Guardian) and as a cheap backup.
- **FactReasoner** — a separate FastAPI service for fact verification (not hosted inside the main backend container).
- **Hosted MCP servers** — CMR, PDS, Code Search, ADS, Astroquery, and the Job Management server — each referenced by env var in the corresponding agents' configs.

See [`deployment.md`](./deployment.md) for the topology in local vs production deployments.

## Streaming model

The contract between backend and frontend is a sequence of typed events. Conceptually these map to the AKD `StreamEvent` family documented in [`../docs/streaming-and-hitl.md`](../docs/streaming-and-hitl.md); on the wire they're serialized as SSE messages with event types like `text-start`, `text-delta`, `data`, and `text-end`. The frontend's stream parser normalizes these into per-node progress updates, token deltas, and structured output drops.

See [`streaming.md`](./streaming.md).

## Persistence

| Kind | Storage |
| --- | --- |
| Workflow configs and latest outputs | Postgres `workflows` table |
| Chat sessions (planner-style) | Postgres `chats` table |
| Single-agent chats | Postgres `agent_chats` table |
| Users | Postgres `users` table |
| LangGraph checkpoints (resume state for paused runs) | Postgres (async `AsyncPostgresSaver`) |
| Rate-limit counters | Redis |

## Related

- [`backend.md`](./backend.md), [`frontend.md`](./frontend.md), [`api-overview.md`](./api-overview.md), [`streaming.md`](./streaming.md), [`deployment.md`](./deployment.md).
- [`../agents/`](../agents/) — the published agents Flow hosts.
- [`../labs/`](../labs/) — the complementary agent playground.
