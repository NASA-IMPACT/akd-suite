# Flow — Deployment

High-level topology of how Flow is deployed locally and in production.

## Local development (Docker Compose)

The simplest path: `docker compose up` against `akd-framework`'s stack. This brings up:

| Service | Port (host) | Purpose |
| --- | --- | --- |
| `postgres` | `5432` | Workflow / chat storage + LangGraph checkpoints |
| `redis` | `6379` | Rate-limiting cache |
| `migrate` | (one-shot) | Alembic migrations on boot |
| `searxng` | `8080` | Meta-search engine for tools that perform web search |
| `ollama` | `11434` | Local LLM runtime (used by Granite Guardian and as cheap fallback) |
| `factuality` | `8000` | Separate FactReasoner FastAPI service for fact verification |
| `api` | `8001` | Main FastAPI backend |

The frontend (`ad-nextjs-ui`) is typically run separately via `npm run dev` on port `3000`, pointed at `http://localhost:8001` through `NEXT_PUBLIC_BACKEND_URL`.

### Getting started

```bash
# Backend
git clone https://github.com/NASA-IMPACT/akd-framework.git
cd akd-framework
cp .env.example .env            # fill in OPENAI_API_KEY, MCP URLs / keys, etc.
uv sync
docker compose up

# Frontend (in a separate terminal)
git clone https://github.com/NASA-IMPACT/ad-nextjs-ui.git
cd ad-nextjs-ui
cp .env.local.example .env.local
npm install
npm run dev
```

## Environment variables of note

### Backend (`akd-framework`)

- LLM provider credentials (OpenAI, Anthropic, etc., depending on configuration).
- MCP server URLs and auth keys: `CMR_MCP_URL`, `PDS_MCP_URL`, `PDS_MCP_KEY`, `CODE_SEARCH_MCP_URL`, `CODE_SEARCH_MCP_KEY`, `ADS_SEARCH_MCP_URL`, `CODE_SEARCH_ADS_SEARCH_KEY`, `ASTRO_MCP_URL`, `EXPERIMENT_STATUS_MCP_URL`, `EXPERIMENT_STATUS_MCP_KEY`.
- Database / cache: `DATABASE_URL`, `REDIS_URL`.
- Admin bootstrap: credentials for the first admin user.
- FactReasoner endpoint (when running in a separate container).

### Frontend (`ad-nextjs-ui`)

- `NEXT_PUBLIC_BACKEND_URL` — backend base URL.
- `NEXT_PUBLIC_FACTUALITY_PIPELINE_ENDPOINT` — FactReasoner URL (if enabled).

See each upstream repo's `.env.example` for the complete and current list.

## Production topology

The upstream repo ships with:

- **AWS CDK** templates (`cdk.json` and companion Python code) for deploying the backend to AWS — typically ECS Fargate behind an Application Load Balancer, with RDS PostgreSQL and ElastiCache Redis. The frontend is usually fronted by CloudFront + S3 or an ECS-served Next.js.
- **Helm charts** (`helm/`) for Kubernetes-based deployments.

The specific hosting of `flow.akd.odsi.io` and `labs.akd.odsi.io` is managed by NASA-IMPACT infrastructure; see the repos for ops-facing detail.

## Scaling considerations

- **Backend is CPU-bound on LLM routing**. Horizontal scaling via additional ECS tasks is straightforward.
- **PostgreSQL** is the most common bottleneck (LangGraph checkpoint writes are frequent). Right-size the instance.
- **Redis** usage is light.
- **SearxNG** and **Ollama** are supplementary; `searxng` is mostly stateless and easy to scale; Ollama is heavier due to model weights.
- **Hosted MCP servers** are independent services and scale separately — they are not part of this stack.

## Where to read next

- [`backend.md`](./backend.md) — what runs inside the `api` container.
- [`frontend.md`](./frontend.md) — what the frontend needs.
- Upstream ops detail: [`NASA-IMPACT/akd-framework`](https://github.com/NASA-IMPACT/akd-framework) and [`NASA-IMPACT/ad-nextjs-ui`](https://github.com/NASA-IMPACT/ad-nextjs-ui).
