# akd-ext — Installation

## From source (current active branch)

```bash
pip install git+https://github.com/NASA-IMPACT/akd-ext.git@develop
```

Or with `uv` (recommended):

```bash
uv pip install git+https://github.com/NASA-IMPACT/akd-ext.git@develop
```

## Local development

```bash
git clone https://github.com/NASA-IMPACT/akd-ext.git
cd akd-ext
git checkout develop
uv venv --python 3.12
uv sync
```

## Key runtime dependencies

- `akd` (akd-core) at a pinned git rev — the base framework.
- `openai-agents` — OpenAI Agents SDK (runtime for `OpenAIBaseAgent`).
- `fastmcp` — MCP server framework used by in-process MCP tools.
- `PyGithub` — used by `RepositorySearchTool` and related tools.
- `pydantic>=2.10.3` — schema validation (inherited from akd-core).

## Dev dependencies

- `pytest>=8.0.0`, `pytest-asyncio` — test framework.
- `pre-commit` — formatting and linting hooks.

## Python version

akd-ext targets Python 3.12.

## Deployed versions

The Flow backend (`akd-framework`) pins a specific git rev of both `akd` and `akd-ext` via its `pyproject.toml`. When a new agent or tool lands in akd-ext, picking it up in Flow requires bumping the pinned rev and redeploying. See [`labs/publishing-to-flow.md`](../../labs/publishing-to-flow.md) for the current promotion story.
