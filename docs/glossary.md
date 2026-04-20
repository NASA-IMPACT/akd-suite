# Glossary

Acronyms and terms that appear throughout the AKD documentation.

## Programs and organizations

- **AKD** — Accelerated Knowledge Discovery. This program.
- **NASA-IMPACT** — NASA's Inter-agency Implementation and Advanced Concepts Team.
- **NASA SMD** — NASA Science Mission Directorate, the organizational umbrella for Earth Science, Planetary Science, Astrophysics, Heliophysics, and Biological and Physical Sciences divisions.

## Products in the AKD ecosystem

- **AKD Flow** — the public multi-agent workflow product: `flow.akd.odsi.io` (backend from `akd-framework`, frontend from `ad-nextjs-ui`).
- **AKD Labs** — the agent playground and benchmarking tool: `labs.akd.odsi.io`.

## Frameworks

- **akd-core** — foundational Python package (`akd`) defining base abstractions: `BaseAgent`, `BaseTool`, `InputSchema`, `OutputSchema`, `StreamEvent`, `HumanTool`, guardrail protocol.
- **akd-ext** — domain extensions (`akd_ext`): concrete agents, tools, MCP glue, OpenAI Agents SDK integration.

## Key framework concepts

- **MCP** — Model Context Protocol. A standard for hosting tools that an LLM agent can call. Most AKD data-discovery agents talk to their upstream APIs via hosted MCP servers.
- **HITL** — Human-in-the-Loop. A design pattern where an agent pauses to request input, approval, or clarification from a human before continuing.
- **CARE** — Collaborative Agent Reasoning Engineering. A NASA-IMPACT methodology used by several AKD agents (CMR CARE, Code Search CARE). Emphasizes transparency, human control, and non-prescriptive output.
- **Guardrail** — an input or output filter applied to an agent. AKD's guardrails are composable via `&` (AND), `|` (OR), and `>>` (fail-fast) operators.

## Data systems queried by AKD agents

- **CMR** — Common Metadata Repository. NASA's metadata catalog for Earth science data; base URL `cmr.earthdata.nasa.gov`.
- **GCMD** — Global Change Master Directory. NASA's controlled vocabulary for Earth science keywords.
- **PDS** — Planetary Data System. NASA's archive for planetary science data, organized into discipline nodes.
  - **GEO** — Geosciences Node (Mars, etc.)
  - **IMG** — Imaging Node
  - **RMS** — Ring-Moon Systems Node
  - **SBN** — Small Bodies Node
  - **PPI** — Planetary Plasma Interactions Node
  - **ATM** — Atmospheres Node
- **PDS3 / PDS4** — the two current archive format generations used across PDS. PDS4 is the modern standard; PDS3 persists for legacy data.
- **ADS** — NASA Astrophysics Data System. Literature search and citation database for astronomy and astrophysics.
- **SDE** — Science Discovery Engine. NASA's institutional search over reports, mission documentation, and technical pages.
- **ASCL** — Astrophysics Source Code Library. Registry of astronomy-related research codes with canonical URLs.
- **MAST** — Mikulski Archive for Space Telescopes (HST, JWST, TESS, Kepler, GALEX).
- **HEASARC** — High Energy Astrophysics Science Archive Research Center (Chandra, XMM, Swift, Fermi, NICER, NuSTAR).
- **IRSA** — NASA/IPAC Infrared Science Archive (Spitzer, WISE, 2MASS, IRAS).
- **SIMBAD** — astronomical object database and name resolver (operated by CDS).
- **VizieR / NED / Gaia / TAP / SIA / SSA / VO** — other astronomical catalog and access protocols referenced by the Astro Search Agent.

## Published AKD agents (as registered at Flow backend startup)

- **CMR CARE Agent** — NASA Earth science dataset discovery.
- **PDS Search Agent** — Planetary Data System discovery across node services.
- **Code Search CARE Agent** — scientific code repository discovery.
- **Astro Search Agent** — astrophysics dataset discovery across NASA archives.
- **Gap Agent** — research-gap detection and synthesis from a corpus of papers (Stage 1 of the CM1 closed-loop pipeline).
- **Capability & Feasibility Mapper** — Stage 2 of CM1: maps research questions to model/compute feasibility.
- **Workflow Spec Builder** — Stage 3 of CM1: designs CM1/WRF experiment workflow specs.
- **Experiment Implementation** — Stage 4 of CM1: translates specs into job-submission payloads.
- **Research Report Generator** — Stage 5 of CM1: produces publication-style reports from completed runs.

## Framework-level guardrails

- **Granite Guardian** — IBM Granite Guardian-based content moderation (single-risk and multi-risk modes, runs via Ollama).
- **Risk Agent** — LLM-judge that evaluates outputs against IBM Risk Atlas and Science Literature Risk taxonomies via DAG-based, importance-weighted criteria.

## Models referenced

- **CM1** — Cloud Model 1, an atmospheric simulation model used as the target for the closed-loop CM1 pipeline (hurricane and convection cases).
- **WRF / HWRF / OLAM** — other atmospheric models referenced in the feasibility mapper.

## Other terms

- **Collection / Granule** — CMR terminology. A *collection* is a dataset-level concept; a *granule* is an individual data file within a collection.
- **LID / LIDVID / PRODUCT_ID / OPUS_ID / ODE_ID** — PDS identifier formats.
- **bibcode** — ADS's unique paper identifier (e.g., `2020ApJ...890...85W`).
- **SSE** — Server-Sent Events. The HTTP streaming protocol Flow uses to push workflow events to the frontend.
- **LiteLLM** — a multi-provider LLM SDK used by akd-core to stay model-agnostic.
- **LangGraph** — a stateful graph-based agent orchestration library used by akd-framework.
