# Closed-Loop CM1 Pipeline

The **Closed-Loop CM1 pipeline** is a five-stage AKD workflow for atmospheric simulation research using the **CM1** model (and, in some stages, **WRF**). It takes a user from a research question, through feasibility mapping and experiment-spec generation, to actually submitting experiment jobs and producing a publication-style report — with **explicit human approval gates at every stage**.

## The five stages

| Stage | Agent | Role |
| --- | --- | --- |
| **1** | [Gap Agent](../gap/) | Identify research gaps, contradictions, and candidate research questions from a paper corpus. Lives outside this directory at [`agents/gap/`](../gap/) since it is also usable standalone. |
| **2** | [capability-feasibility-mapper/](./capability-feasibility-mapper/) | Evaluate whether the research questions and hypotheses can be realistically tested using CM1/WRF and the cluster. Produce a feasibility assessment with evidence paths. |
| **3** | [workflow-spec-builder/](./workflow-spec-builder/) | Design a single executable experiment workflow specification (CM1 or WRF, never both) with baseline + perturbations. |
| **4** | [experiment-implementation/](./experiment-implementation/) | Translate the Stage-3 spec into structured FileEdit JSON and submit the experiment batch as a job via MCP. |
| **5** | [research-report-generator/](./research-report-generator/) | Check job completion, fetch result figures, and produce a publication-style scientific report. |

The akd-ext codebase also includes an `InterpretationPaperAssemblyAgent` class targeting CM1 output interpretation + notebook + paper generation. It is **not currently registered at Flow backend startup** as a published agent (it lives in the code but is not exposed through Flow), so it is intentionally not documented here as a published CM1 pipeline stage. If/when it becomes Flow-published, it will gain a directory here.

## Published-to-Flow note

All four sub-agents (and Gap at Stage 1) are registered at Flow backend startup. However, their own descriptions flag them as `Stage-N: INTERNAL ONLY — Do NOT select this agent in planner workflows. It is part of a specialized pipeline and cannot be used standalone.` This means:

- They are **individually addressable** via Flow's single-agent chat endpoint.
- They are **not** meant to be chosen à la carte by the planner when composing workflows from a natural-language request. The CM1 pipeline is an end-to-end story, not a pick-and-mix menu.

## Shared context

Several stages embed the same two context markdowns in their instructions:

- `cm1_readme.md` — CM1 model documentation including namelist reference and capabilities. Used by Stages 2, 3, and 4.
- `cluster_it.md` — UAH-NSSTC (Matrix) cluster documentation covering queues, QOS, pre-emption, job scheduling, and caveats. Used by Stage 2.

These are shipped alongside the agents in akd-ext; they are not pulled at runtime but baked into the agent's instructions at construction time.

## Sections

- [`pipeline-overview.md`](./pipeline-overview.md) — narrative of how the five stages chain together and what each hand-off contains.
- Per-stage directories — each with a `README.md` and an `artifacts/` tree mirroring the published-agent pattern.

## Deeper reading

- Upstream: [`akd_ext/agents/closed_loop_cm1/`](https://github.com/NASA-IMPACT/akd-ext).
- Stage 1: [`agents/gap/`](../gap/).
