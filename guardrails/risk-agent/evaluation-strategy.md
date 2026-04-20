# Risk Agent — Evaluation Strategy

A more detailed walkthrough of how the Risk Agent evaluates an output, for readers who want to understand what the "DAG-based importance-weighted evaluation" actually does.

## The overall flow

```
┌──────────────────────────────────────────────────────────────┐
│ 1. Criteria Generation                                       │
│                                                              │
│   For each configured risk category, the Risk Agent uses     │
│   a general-purpose LLM to generate a set of specific        │
│   evaluation criteria tailored to that risk.                 │
└────────────────┬─────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ 2. DAG Construction                                          │
│                                                              │
│   Build a hierarchical DAG:                                  │
│                                                              │
│   Level 0 — one criterion node per criterion                 │
│     (boolean-ish verdicts, each with HIGH/MEDIUM/LOW         │
│      importance label)                                       │
│                                                              │
│   Level 1 — one aggregation node per risk category           │
│     (importance-weighted combine across criteria for         │
│      that category; HIGH criteria weighted heavier)          │
│                                                              │
│   Top level — one final summary node                         │
│     (combine per-category aggregations into an               │
│      overall pass/fail + score)                              │
└────────────────┬─────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ 3. Evaluation                                                │
│                                                              │
│   DeepEval's DAGMetric.a_measure() runs each node            │
│   against the candidate output (and optional context /       │
│   source_context) producing verdicts and scores.             │
└────────────────┬─────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ 4. Verdict Extraction                                        │
│                                                              │
│   Read verdicts from the DAG nodes' outputs (primary         │
│   path) or parse verbose logs (fallback). Aggregate          │
│   into detected risks + overall risk score.                  │
└────────────────┬─────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────┐
│ 5. Optional Risk Report                                      │
│                                                              │
│   For failed HIGH-importance criteria, an internal           │
│   _RiskReportAgent synthesizes a human-readable              │
│   explanation summarising why the output failed.             │
└──────────────────────────────────────────────────────────────┘
```

## Why the DAG?

Two concerns drove the design:

- **Specificity**: Rather than asking an LLM "is this hallucinatory?" in the abstract, the agent generates concrete criteria first (e.g., "does the output cite specific sources for numerical claims?" "are any quoted values internally consistent?"). Verdicts on concrete criteria are more reliable than a single abstract judgement.
- **Importance weighting**: Not every criterion is equally important. Marking criteria with HIGH / MEDIUM / LOW importance lets the aggregation reflect severity: one HIGH-importance failure outweighs multiple LOW-importance concerns.

## Pass threshold

The Risk Agent's default `pass_threshold` is **0.9**. The overall score from the DAG must be ≥0.9 for the guardrail to pass. This is a high bar by design — the Risk Agent is meant to be a strict science-literature-quality check. Lower the threshold if you want a more permissive judge, or raise it for a stricter one.

## Risk weights

The `risk_weights` parameter lets callers override per-category weight in the final aggregation. Useful when the default evenly-weighted combination doesn't match the caller's priorities (e.g., "for this agent, I care about `hallucination_identification` twice as much as `overgeneralization`").

## Criterion validation

By default, `validate_categories=True` requires that input categories match the guardrail's configured category type (so you can't accidentally mix `ScienceRiskCategory` and `AtlasRiskCategory` in one call). Set to `False` to allow mixed categories; the implications of mixing are not exhaustively documented upstream, so exercise caution.

## Failure modes to be aware of

- **Criterion generation drift**: Because criteria are generated per run, two runs against the same content may produce different criteria. The DAG structure is deterministic, but the criteria content is not.
- **Cost scaling**: More categories → more criteria → more LLM calls. Budget accordingly.
- **Context window**: Very long outputs combined with many criteria can strain model context windows. Use concise outputs or narrower category sets for long-form content.

## See also

- [`README.md`](./README.md) — overview, configuration, composition patterns.
- [`science-literature-risks.yaml`](./science-literature-risks.yaml) — the Science Literature Risks source.
- [`risk-atlas.yaml`](./risk-atlas.yaml) — the IBM Risk Atlas source.
