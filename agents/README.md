# Agents

This directory catalogs the **agents currently published to Flow** — each entry documents an agent that is registered at the Flow backend on startup and available to users at [flow.akd.odsi.io](https://flow.akd.odsi.io).

Each agent has:

- A top-level `README.md` explaining what it is, who uses it, what data source it targets, and what a successful run looks like.
- An `artifacts/` directory containing the agent's **consumable knowledge layer** — system prompt (redistributed across `contexts/`), and tool descriptions under `tools/`. No code references; the artifacts are designed to be portable context for future akd-ext loaders.

## Published agents

### Data discovery

- [**CMR**](./cmr/) — NASA Earthdata dataset discovery via the Common Metadata Repository. Uses the CARE reasoning loop (clarify → analyze → rank → explain).
- [**PDS**](./pds/) — Planetary Data System discovery across GEO, IMG, RMS, SBN, PPI, and ATM nodes. Catalog-first routing; returns both PDS3 and PDS4 versions when available.
- [**Astro Search**](./astro-search/) — Astrophysics dataset discovery across NASA archives (MAST, HEASARC, IRSA, plus SIMBAD, ADS, Gaia, VizieR, NED). Supports literature-, coordinate-, archive-, and event-driven patterns.
- [**Code Search**](./code-search/) — Scientific code repository discovery across the NASA-verified code corpus, SDE, ADS literature, and supplementary web search. CARE reasoning loop.

### Research synthesis

- [**Gap**](./gap/) — Research-gap detection from a corpus of academic papers. Six-stage pipeline with human approval between stages. Serves as **Stage 1** of the Closed-Loop CM1 pipeline.

### Closed-Loop CM1 pipeline

- [**closed-loop-cm1/**](./closed-loop-cm1/) — a five-stage pipeline for atmospheric simulation research: gap analysis → feasibility mapping → workflow spec → experiment implementation → report generation. See the group's `pipeline-overview.md` for how the stages chain.

## What does NOT live here

- **Framework-level agents** that act as internal judges or guardrail providers (e.g., the Risk Agent) are NOT published to Flow and live under [`guardrails/`](../guardrails/).
- **Planner-internal agents** used by the Flow backend's workflow planner live in `akd-core` and are documented under [`frameworks/akd-core/`](../frameworks/akd-core/).

## How this directory is organized

Each agent's `artifacts/` follows a consistent shape:

```
agents/<agent>/
├── README.md                     # reader-facing one-pager
└── artifacts/
    ├── index.md                  # manifest of what's inside
    ├── contexts/
    │   ├── index.md
    │   ├── role-and-objective.md
    │   ├── reasoning-strategy.md
    │   ├── constraints.md
    │   ├── output-format.md
    │   └── (other context files, only when the agent's source prompt contains the material)
    └── tools/
        ├── index.md
        └── <tool>/index.md       # one directory per tool actually wired in the agent's current config
```

Artifact contents are derived strictly from the current akd-ext system prompt and config of each agent — nothing invented. When a tool is backed by an MCP server, its `index.md` states the `server_label` and the URL env var.
