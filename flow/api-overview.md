# Flow — API Overview

A conceptual map of the Flow backend's HTTP API. For parameter-level detail, consult the backend's generated OpenAPI spec — we deliberately do not duplicate it here because it evolves fast.

All endpoints are prefixed `/api/v1`. All non-auth endpoints require an `Authorization: Bearer <JWT>` header.

## Authentication and users

| Endpoint | Purpose |
| --- | --- |
| `POST /auth/register` | Register a new user (or bootstrap an admin from env on first run). |
| `POST /auth/jwt/login` | Exchange credentials for a JWT access token. |
| `GET /users/{user_id}` | Fetch user profile. |
| `PATCH /users/{user_id}` | Update user profile. |
| `PATCH /admin/users/{user_id}/change_status` | Admin-only user verification status update. |

## Workflows

Workflows are the primary primitive — a graph of agent nodes that can be run, saved, and shared.

| Endpoint | Purpose |
| --- | --- |
| `GET /workflows/` | List the caller's workflows. |
| `POST /workflows/` | Create a new workflow (from a config object). |
| `GET /workflows/{workflow_id}` | Fetch a workflow's config. |
| `POST /workflows/{workflow_id}/run` | Execute the workflow. Returns a streaming response (SSE) of execution events. |
| `GET /workflows/{workflow_id}/retrieve_latest_state` | Fetch the most recent execution output. |
| `GET /workflows/{workflow_id}/workflow_configuration_variables` | Fetch the input schema for the workflow (used by the UI to render parameter forms). |

## Chats (planner)

A chat is a conversational session with the LLM planner that produces and refines a workflow.

| Endpoint | Purpose |
| --- | --- |
| `GET /chats/` | List chats. |
| `POST /chats/` | Start a new chat with a user message; planner returns a workflow blueprint. |
| `GET /chats/{chat_id}` | Fetch chat details and associated workflow. |
| `POST /chats/{chat_id}/continue` | Continue the conversation; planner refines the workflow. |

Chat sharing and import endpoints exist alongside these for making chats public or copying shared chats into a workspace.

## Agent chats (single-agent mode)

A lighter-weight mode where the user talks to one agent directly without composing a full workflow.

| Endpoint | Purpose |
| --- | --- |
| `GET /agent_chats/` | List the caller's single-agent chats. |
| `POST /agent_chats/{agent_name}` | Send a message to a named agent; streams the agent's events. |

`agent_name` corresponds to the agent ids registered at backend startup (e.g., `cmr_care`, `code_search_care`). See [`../frameworks/akd-ext/agents-index.md`](../frameworks/akd-ext/agents-index.md) for the full list.

## Files

| Endpoint | Purpose |
| --- | --- |
| `POST /files/upload` | Upload a file the user wants to attach to an agent run. Returns a reference the UI then attaches to a node's input. |

## Streaming endpoint shape

The workflow run and single-agent chat endpoints are the only endpoints that return Server-Sent Events. Each event is a JSON object with a `type` field (e.g., `text-start`, `text-delta`, `data`, `text-end`) and contextual metadata (which node, which tool call, etc.). See [`streaming.md`](./streaming.md) for the event model.

## Error model

- `401` — missing / invalid JWT.
- `403` — authorized but unverified or lacking permission.
- `404` — resource not found or not owned by the user.
- `429` — rate limit exceeded.
- `5xx` — internal error; inspect server logs.

## Where to read next

- [`streaming.md`](./streaming.md) — the SSE event model in detail.
- [`backend.md`](./backend.md) — how the endpoints are implemented.
- Upstream OpenAPI: available at `/docs` on a running backend instance.
