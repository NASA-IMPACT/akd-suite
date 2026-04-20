# Gap Agent

The **Gap Agent** identifies defensible research gaps, contradictions, and candidate research questions from a user-curated corpus of academic papers. It is **evidence-grounded, non-authoritative, and human-in-the-loop** — a structured evidence synthesizer that never declares novelty, resolves contradictions, or judges scientific importance on its own.

## At a glance

| | |
| --- | --- |
| **Purpose** | From a curated corpus of academic papers, identify research gaps, contradictions, and candidate research questions with full traceability |
| **Users** | Researchers preparing literature reviews, research proposals, or follow-on experiment designs |
| **Data source** | The user-provided corpus (typically 1–50 papers, PDFs or extracted text / summaries) |
| **Division** | Cross-SMD — applicable to any scientific domain |
| **Reasoning pattern** | Six-stage pipeline with explicit human-approval gates between stages |
| **Output** | Structured markdown report: Ranked Gap List, Contradictions / Disagreements, (optional) Research Question Add-On |
| **Source** | [NASA-IMPACT/akd-ext — `akd_ext/agents/gap.py`](https://github.com/NASA-IMPACT/akd-ext) |

## Role in the CM1 pipeline

The Gap Agent is **Stage 1 of the Closed-Loop CM1 pipeline**. Its output (hypotheses and research questions) feeds into the Capability & Feasibility Mapper (Stage 2). The agent's own description flags it as INTERNAL ONLY for planner workflows; its primary use is as the first stage of the CM1 pipeline or as a tool a researcher invokes directly via the single-agent chat interface.

See [`agents/closed-loop-cm1/`](../closed-loop-cm1/) for the full pipeline story.

## What a successful run looks like

The user provides a corpus (or summaries) and optional configuration (e.g., whether to include research question suggestions). The agent then executes six stages in order, pausing for explicit human approval between each:

1. **Scientific Scope Inference** — propose multiple plausible scopes and let the user choose.
2. **Structured Extraction** — read each paper in full and extract claims, evidence, methods, assumptions, and limitations with paragraph-level traceability.
3. **Gap-Matrix Proposal** — propose 3–4 alternative analytical lenses (methods / data / regimes / theory).
4. **Gap Identification** — surface explicit gaps (author-stated), inferred gaps (cross-paper synthesis), and contradictions.
5. **Research Question / Hypothesis Suggestions** (optional, enabled by default) — 6–10 descriptive/explanatory questions linked to specific gaps.
6. **Qualitative Prioritization** — tiered clusters (High / Medium / Exploratory) with no numeric scoring.

## What it never does

- Declares novelty
- Resolves scientific contradictions
- Judges feasibility, importance, or significance
- Assumes scope elements without evidence
- Silently introduces assumptions
- Moves to the next stage without user confirmation

## No configured tools

Unlike the data-discovery agents (CMR, PDS, Code Search, Astro), the Gap Agent has **no MCP tools** wired into its config. It is a pure reasoning agent that operates over the provided paper content — each stage runs entirely within the LLM context. Its artifact directory therefore has no `tools/` subdirectory.

## Deeper reading

- [`artifacts/`](./artifacts/) — the agent's full role, constraints, six-stage process, and output format.
- Upstream implementation: [`akd_ext/agents/gap.py`](https://github.com/NASA-IMPACT/akd-ext).
- [`agents/closed-loop-cm1/pipeline-overview.md`](../closed-loop-cm1/pipeline-overview.md) — how Gap feeds the CM1 pipeline.
