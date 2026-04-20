# Capability & Feasibility Mapper (CM1 Stage 2)

The **CARE Capability & Feasibility Mapper** evaluates whether research questions and hypotheses (from Stage 1, the Gap Agent) can be realistically tested using available numerical models (CM1, WRF, HWRF, OLAM), codebases, and cluster resources. It produces a structured feasibility assessment with evidence-backed reasoning — but never makes final decisions to run experiments.

## At a glance

| | |
| --- | --- |
| **Stage** | 2 (in the CM1 closed-loop pipeline) |
| **Purpose** | Produce a structured capability / feasibility assessment with evidence paths, compute estimates, cluster-fit, risks, and scoring |
| **Expertise** | Numerical weather prediction (especially CM1, also WRF/HWRF/OLAM), model documentation, HPC compute estimation, feasibility evaluation |
| **Input** | `research_questions_md` — markdown string from the Gap Agent with research questions, hypotheses, variables, and causality guardrails |
| **Output** | `report` (markdown feasibility report), `feasibility_score` (float 0–1), `can_proceed` (boolean), `unresolved_items`, `next_actions` |
| **Tools** | None wired (pure reasoning agent); embeds `cm1_readme.md` and `cluster_it.md` as instruction context |
| **Source** | [`akd_ext/agents/closed_loop_cm1/capability_feasibility_mapper.py`](https://github.com/NASA-IMPACT/akd-ext) |

## What a successful run produces

A structured feasibility assessment that:

- Parses each research question / hypothesis
- Extracts required capabilities (dynamics/numerics, physics schemes, boundary/initial conditions, coupling, diagnostics, resolution/scale)
- Retrieves evidence from CM1 documentation, citing **specific namelist parameters by name and value** (e.g., `isnd=7`, `output_cape=1`)
- Computes compute feasibility from config parameters (grid size, integration time, timestep, storage, memory)
- Assesses cluster fit separately from compute estimate (queues, QOS, pre-emption, memory, walltime)
- Identifies risks
- Scores overall feasibility (1=blocked … 5=clearly feasible) with confidence adjustments
- Builds global summary and per-hypothesis matrices
- Lists unresolved items and suggested next actions

The assessment **never approves experiments**. It includes the disclaimer: *"This report provides capability/feasibility assessments and evidence paths only. It does not make a final decision to run experiments; human approval is required."*

## Deeper reading

- [`artifacts/`](./artifacts/) — full knowledge layer.
- Upstream: [`akd_ext/agents/closed_loop_cm1/capability_feasibility_mapper.py`](https://github.com/NASA-IMPACT/akd-ext).
- [`../pipeline-overview.md`](../pipeline-overview.md) — where this agent fits in the CM1 pipeline.
