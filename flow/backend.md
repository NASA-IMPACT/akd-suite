# Flow — Backend

A high-level overview of [`akd-services`](https://github.com/NASA-IMPACT/akd-services), the FastAPI + LangGraph backend that powers Flow.

## Tech stack

- **Python 3.12** with `uv` for dependency management.
- **FastAPI 0.109+** with `uvicorn` for the API surface.
- **SQLAlchemy 2.0+** (async, via `asyncpg`) on **PostgreSQL 15** for persistence.
- **LangGraph 0.6+** for agent workflow orchestration and a PostgreSQL-backed checkpointer for resuming paused runs.
- **Redis 7** + **FastAPI-Limiter** for rate limiting.
- **fastapi-users** + **JWT** (via `python-jose`, `bcrypt`) for authentication.
- **LiteLLM** for provider-agnostic LLM calls; **Ollama** as an optional local runtime.
- **SearxNG** for web search (used by tools that need general web search).
- **FactReasoner** (separate FastAPI service) for fact verification.

Runtime dependencies include pinned git revs of `akd` (akd-core) and `akd-ext` — when new agents or tools are added to akd-ext, the backend picks them up by bumping the pinned rev and redeploying. See [`../labs/publishing-to-flow.md`](../labs/publishing-to-flow.md).

## Top-level layout

The backend source lives under `src/api/runtime/app/` in the upstream repo, organized roughly as:

- `main.py` — FastAPI app, lifespan hooks, agent registration at startup.
- `api/` — routers (auth, users, workflows, chats, agent_chats, files, admin).
- `services/` — factory code for agents and LangGraph nodes, workflow graph construction, per-agent guardrail config.
- `models/` — SQLAlchemy ORM models (workflows, chats, agent_chats, users).
- `schemas/` — Pydantic response / request models (workflow, chat, planner).
- `db/` — async session and checkpointer setup.
- `core/config.py` — environment-driven settings.

## Startup behavior

On lifespan startup, `main.py`:

1. Connects to Postgres and Redis.
2. Initializes LangGraph's `AsyncPostgresSaver` as the shared workflow checkpointer.
3. Registers each published AKD agent with the agent registry by calling `registry.register_agent(agent_class=..., agent_id=...)`.
4. Unregisters retired agent ids (legacy planners had different names).
5. Starts the FastAPI app.

The authoritative registration list is in the upstream `main.py`. It is mirrored in `akd-suite` at [`../frameworks/akd-ext/agents-index.md`](../frameworks/akd-ext/agents-index.md).

## Request lifecycle

### Workflow execution (`POST /api/v1/workflows/{workflow_id}/run`)

1. Authenticates the user (JWT).
2. Applies rate limiting.
3. Loads the workflow's current config from Postgres.
4. Calls `services/akd_workflow.py` to build a LangGraph graph from the config's nodes and edges.
5. Each node is a wrapper (`SingleAgentNodeTemplate`) around an agent instance produced by `services/agent_factory.py` / `services/node_factory.py`, with optional guardrails applied based on `services/guardrails_config.py`.
6. Returns a `StreamingResponse` that yields Server-Sent Events as the graph executes.
7. On pause (HITL), LangGraph checkpoints state to Postgres; on resume, the backend replays from the checkpoint and injects the human response.
8. On completion, the final outputs are stored in the workflow's `latest_output` column.

### Planner chat (`POST /api/v1/chats/`, `POST /api/v1/chats/{chat_id}/continue`)

1. The user's message is sent to the planner agent (`akd.planner.llm_planner`) with the registry of available agent ids.
2. The planner returns a `PlannerWorkflowConfig` — nodes + edges + suggested parameters.
3. The backend saves the chat and optionally creates a workflow record linked to it.

### Single-agent chat (`POST /api/v1/agent_chats/{agent_name}`)

1. Instantiates the named agent directly.
2. Streams the agent's `astream()` events as SSE.
3. Persists the chat thread per user.

## Agent registry and factories

- `services/agent_factory.py` maps `agent_id` strings to an instance builder (constructor + input schema). Adding a new agent means registering it in `main.py` and (where needed) teaching the factory how to construct it.
- `services/node_factory.py` wraps an agent as a LangGraph node template with guardrails, debug flags, and node config merged in.
- `services/guardrails_config.py` carries per-agent guardrail specifications (e.g., "wrap CMR Agent's output with Granite + Risk Agent composed fail-fast").

## Persistence

| Entity | Table | Notes |
| --- | --- | --- |
| Users | `users` | via `fastapi-users` |
| Workflows | `workflows` | stores the workflow's `config` (graph) and `latest_output` |
| Chats | `chats` | planner conversations; each has an associated workflow |
| Agent chats | `agent_chats` | single-agent direct chats |
| LangGraph checkpoints | Postgres (via `AsyncPostgresSaver`) | resume state for paused runs |

## Where to read next

- [`api-overview.md`](./api-overview.md) — the conceptual API surface.
- [`streaming.md`](./streaming.md) — the SSE event model.
- [`deployment.md`](./deployment.md) — how the backend is deployed (Docker, CDK, Helm).
- [`../frameworks/akd-ext/`](../frameworks/akd-ext/) — what the backend imports to define agents.
- Upstream repo: [`NASA-IMPACT/akd-services`](https://github.com/NASA-IMPACT/akd-services).
