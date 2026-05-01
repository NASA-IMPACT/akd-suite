# Risk Agent

The **Risk Agent** is AKD's framework-level LLM-judge guardrail. It evaluates agent outputs against science-oriented and enterprise AI risk taxonomies using a dynamically-generated, importance-weighted evaluation graph. It is a **standalone, reusable guardrail** — not a user-facing "published agent," so it lives here under `guardrails/` rather than under `agents/`.

## What it is

A provider in `akd-core` (`akd.guardrails.providers.risk_agent`) implementing the standard `GuardrailProtocol`. Primarily designed to **guard agent outputs** — wrapping other agents via `apply_guardrails(output_guardrail=<RiskAgent instance>)` or the `@guardrail` decorator.

## How it works (high level)

Unlike Granite Guardian, the Risk Agent does not use a fixed-purpose classification model. For each configured risk category, it:

1. Dynamically generates evaluation criteria for that risk using a general-purpose LLM (via the shared `get_response_async()` path inside akd-core).
2. Builds a hierarchical **DAGMetric** (Deep Acyclic Graph) from those criteria:
   - **Level 0 — criterion nodes**: Individual evaluators that check specific criteria.
   - **Level 1 — aggregation nodes**: Per-risk importance-weighted aggregation where HIGH criteria get heavier weight.
   - **Top level — final summary node**: Overall risk assessment that merges all categories.
3. Runs the evaluation using DeepEval's `DAGMetric.a_measure()`.
4. Extracts verdicts from DAG node outputs (primary) or verbose logs (fallback).
5. Optionally generates a detailed risk report for failed HIGH-importance criteria, produced by an internal helper agent (`_RiskReportAgent`).

See [`evaluation-strategy.md`](./evaluation-strategy.md) for a deeper walkthrough of the DAG structure and importance weighting.

## Why it exists

Granite Guardian is strong at general content moderation (harm, bias, jailbreak, etc.), but scientific outputs have domain-specific failure modes — hallucination, attribution gaps, consistency failures, overgeneralization, positivity bias, multidisciplinary gaps — that don't map neatly onto harm categories. The Risk Agent's taxonomy is designed for that: a Science Literature Risk set plus the IBM Risk Atlas for enterprise AI risks. Both are YAML-driven and mirrored here:

- [`science-literature-risks.yaml`](./science-literature-risks.yaml) — the source for the `ScienceRiskCategory` enum.
- [`risk-atlas.yaml`](./risk-atlas.yaml) — the source for the `AtlasRiskCategory` enum.

## Default configuration

- Defaults to evaluating a subset of `ScienceRiskCategory`. Callers override via the `risk_categories` parameter.
- `pass_threshold` defaults to **0.9** — the overall score must be ≥0.9 for the output to pass.
- `risk_weights` allows overriding per-category importance.
- `dag_verbose=True` emits detailed step logs.
- Risk reports auto-generate for failed HIGH criteria (configurable).

## Where to apply it

Common patterns:

- **Output-only guarding**: `apply_guardrails(agent, output_guardrail=RiskAgent(risk_categories=[...]))`
- **Fail-fast composition with Granite**: `granite >> risk_agent` — cheap moderation first, then the expensive LLM judge only when Granite passes.
- **Per-agent tuning**: for agents in scientific workflows (CMR, PDS, Code Search, Astro, Gap, CM1 stages), tune `risk_categories` to the failure modes that matter for that domain (e.g., CMR might emphasize `consistency` and `attribution`; Code Search might emphasize `verification` and `hallucination_identification`).

## Practical caveats

- **Cost**: Every run triggers LLM calls to generate criteria plus DAG evaluation. Multi-turn conversations with many categories can be expensive.
- **Latency**: DAG evaluation is not instant. Use sparingly on latency-sensitive paths; or combine `>>` with a cheap filter ahead.
- **Model choice**: The default uses Claude via `get_response_async()`. Model selection follows akd-core's LLM-agnostic conventions.

## Companion docs

- [`evaluation-strategy.md`](./evaluation-strategy.md) — DAG-based evaluation walkthrough.
- [`risk-atlas.yaml`](./risk-atlas.yaml) — verbatim copy of akd-core's IBM Risk Atlas source.
- [`science-literature-risks.yaml`](./science-literature-risks.yaml) — verbatim copy of akd-core's Science Literature Risks source.

## Upstream

Implementation: `akd.guardrails.providers.risk_agent` in [NASA-IMPACT/akd-core](https://github.com/NASA-IMPACT/akd-core). Category loaders: `akd.guardrails.categories.atlas`.
