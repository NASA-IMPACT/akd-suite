# Labs — Architecture

[`NASA-IMPACT/akd-labs`](https://github.com/NASA-IMPACT/akd-labs) is a multi-tenant web application for LLM agent benchmarking and experimental agent development.

## Tech stack

- **FastAPI** backend (Python).
- **Next.js** frontend (TypeScript / React).
- **PostgreSQL** for tenants, users, projects, agents, traces, runs, results.
- **Docker Compose** for local development (database + API + UI + nginx).
- Built on top of **OpenAI Agents SDK** (`openai-agents>=0.0.7`) for agent execution.

## Multi-tenancy model

- **Organizations** — the top-level tenant.
- **Projects** — scoped within an organization. Most resources (agents, runs) live at project scope.
- **Users** — belong to one or more organizations with role-based permissions.
- **RBAC** — role-based access control with project- and organization-level permission checks. See `services/permissions.py` in the upstream repo.

## Core entities

- **AgentConfig** — an authored agent: `name`, `description`, `system_prompt`, `model` (string), `source_code` (optional Python source, AST-parsed for metadata extraction), `tools_config` (JSON), `model_settings` (JSON), `visibility_scope` (`project` or `organization`), `created_by_user_id`, `published_by_user_id`.
- **Run** — a benchmark run executing an agent against a suite; records results, cost, and grading.
- **TraceLog** — captured per chat turn: full request/response, tool calls, reasoning, tokens, cost estimate.
- **Result** — a graded row from a run.

## High-level layout

The codebase is organized roughly as:

- `api/` — FastAPI routers: `agents.py` (CRUD + chat), `runs.py` (benchmarking), `traces.py` (observability), `export.py` (CSV/JSON/HTML result export), etc.
- `models/` — SQLAlchemy ORM for the entities above.
- `services/` — core logic: permissions, tenancy, agent execution orchestration, pricing estimates.
- `executors/` — an `AgentExecutor` interface with built-in implementations (OpenAI Agents SDK executor, registry resolution).
- `workers/` — background job execution plus an SSE event bus for streaming to the UI.
- `frontend/src/` — the Next.js App Router for the lab's UI (agents, runs, traces, settings).
- `schemas/` — Pydantic request / response models.
- `docs/ARCHITECTURE.md` — system design doc in the upstream repo.

## Pricing and cost tracking

Labs computes an estimated cost per trace turn from token usage and model pricing tables. The estimate is approximate but useful for A/B comparing agent configurations.

## Observability

Every agent run generates a full trace log with:

- System prompt as sent.
- Each tool call's name, input, and output.
- Model reasoning summary (when enabled).
- Token counts (input / output / cached / reasoning).
- Per-turn cost estimate.
- Wall-clock time.

## Relationship to Flow

- **Labs and Flow are separate deployments** — different apps, different domains (`labs.akd.odsi.io` vs `flow.akd.odsi.io`), different databases.
- **Agents do not live in both at runtime.** An agent developed in Labs is validated there; to appear in Flow, it's written into `akd-ext` and picked up via a backend redeploy. See [`publishing-to-flow.md`](./publishing-to-flow.md).
- **Labs imports `openai-agents` directly**; it does not (today) import `akd-core` / `akd-ext` as libraries. The agent primitives used in Labs are simpler — the Labs UI stores a system prompt, model id, and tools config rather than the full akd-ext `OpenAIBaseAgent` subclass.

## Where to read next

- [`user-guide.md`](./user-guide.md) — using the UI.
- [`agent-development-lifecycle.md`](./agent-development-lifecycle.md) — create → chat-test → trace → iterate → publish.
- [`benchmarking.md`](./benchmarking.md) — benchmark runs and result grading.
- [`publishing-to-flow.md`](./publishing-to-flow.md) — current and planned paths from Labs to Flow.
- Upstream: [`NASA-IMPACT/akd-labs`](https://github.com/NASA-IMPACT/akd-labs).
