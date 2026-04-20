# Guardrails

AKD's guardrail system is a pluggable, composable layer that wraps agent inputs or outputs with safety and risk checks. Guardrails live at the **framework level** — they are reusable across any agent, including agents you build yourself.

## The architecture

A guardrail is anything that implements the `GuardrailProtocol`: given a `GuardrailInput` (content plus optional context), produce a `GuardrailOutput` describing detected risks. Both sync `check()` and async `acheck()` are supported.

### Applying guardrails

Two entry points:

- A **decorator**, `@guardrail(input_guardrail=..., output_guardrail=...)`, applied to an agent class.
- A **runtime wrapper**, `apply_guardrails(agent, input_guardrail=..., output_guardrail=...)`, applied to an agent instance.

Either way, the agent's response gets wrapped with a mixin that attaches `input_guardrail_result` and `output_guardrail_result`. A computed `guardrails_passed` flag lets callers short-circuit on failures.

### Composing guardrails

Guardrails support Python operators for composition:

- `g1 & g2` — **AND**: both must pass. Runs in parallel; merges detected risks; fails if any detects risk.
- `g1 | g2` — **OR**: at least one must pass. Runs in parallel; passes if any passes; reports risks only if all fail.
- `g1 >> g2` — **fail-fast**: runs sequentially (e.g., cheap filter first, expensive judge second); returns immediately on first failure.

These operators produce a `CompositeGuardrail` under the hood, with modes `ALL`, `ANY`, and `FAIL_FAST`.

### Configuration

AKD reads global guardrail defaults from `akd.configs.project.GuardrailSettings` (environment-driven, e.g., `GUARDRAILS__FAIL_ON_INPUT_RISK`). Per-agent overrides are supported through the decorator arguments.

## The providers

AKD ships two complementary guardrail providers:

- [**Granite Guardian**](./granite/) — IBM Granite Guardian-based content moderation. Fast, opinionated, content-focused. Covers jailbreak, violence, sexual content, profanity, social bias, unethical behavior, harm, plus RAG-specific checks (groundedness, relevance, answer relevance). Runs via Ollama.
- [**Risk Agent**](./risk-agent/) — LLM-judge that evaluates outputs against the IBM Risk Atlas and a NASA-IMPACT Science Literature Risk taxonomy. Slower but domain-aware; detects hallucination, attribution gaps, consistency issues, overgeneralization, outdated confidence, multidisciplinary failures, and more. Uses a DAG-based, importance-weighted evaluation graph.

### When to use which

| You need… | Use |
| --- | --- |
| Input-side content moderation (prompt injection, harmful requests) | Granite Guardian (single-risk or multi-risk) |
| Fast, low-cost output filtering for clearly harmful content | Granite Guardian |
| Science-specific output quality assessment (hallucination, attribution, consistency) | Risk Agent |
| Full-stack safety + science-risk evaluation | Granite Guardian **>>** Risk Agent (fail-fast) |

## What this directory is NOT

- These are **not** published-agent artifacts. Guardrail providers are reusable framework components; they have no `artifacts/` subtree (contrast with [`agents/`](../agents/)).
- These are **not** the place for agent-specific operational constraints — those live inside each agent's `artifacts/contexts/constraints.md`. The distinction: an agent's constraints are behavioral rules the LLM follows (e.g., "CMR agent must never recommend datasets"); a guardrail is a separate filter that can veto or flag a response after the fact.

## Further reading

- [`granite/README.md`](./granite/README.md), [`granite/risk-categories.md`](./granite/risk-categories.md), [`granite/harm-categories.md`](./granite/harm-categories.md), [`granite/usage.md`](./granite/usage.md).
- [`risk-agent/README.md`](./risk-agent/README.md), [`risk-agent/evaluation-strategy.md`](./risk-agent/evaluation-strategy.md), [`risk-agent/risk-atlas.yaml`](./risk-agent/risk-atlas.yaml), [`risk-agent/science-literature-risks.yaml`](./risk-agent/science-literature-risks.yaml).
