# akd-core — Installation

## From source (current active branch)

```bash
pip install git+https://github.com/NASA-IMPACT/akd-core.git@develop
```

## With `uv` (recommended for AKD development)

```bash
uv pip install git+https://github.com/NASA-IMPACT/akd-core.git@develop
```

## Key runtime dependencies

- `pydantic>=2.10.3` — schema validation.
- `litellm` — multi-provider LLM SDK.
- `langgraph` — graph-based agent orchestration (used by Flow's backend).
- `langchain-*` — integration libraries.
- `instructor` — structured output support via LLM.
- `crawl4ai`, `markdownify`, `pymupdf`, `lxml-html-clean` — scraping and document parsing.
- `requests`, `httpx` — HTTP clients.

## Local development

```bash
git clone https://github.com/NASA-IMPACT/akd-core.git
cd akd-core
git checkout develop
uv venv --python 3.12
uv sync
```

See the upstream README for test-running and contribution guidelines.

## Python version

akd-core targets Python 3.12.
