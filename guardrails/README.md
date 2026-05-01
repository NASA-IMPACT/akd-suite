# Guardrails

## What this is

The AKD guardrail layer is a **composable safety filter** that sits between an agent and the world. It does not change how an agent reasons — it inspects what goes in and what comes out, and gives the system a chance to flag, score, or veto content before it reaches a user (or before the agent acts on a malformed prompt).

Guardrails are deliberately separate from the agents themselves. An agent's own behavioral rules — "the CMR agent never recommends a dataset," "the astro-search agent never fabricates obs IDs" — live inside that agent's prompt as `artifacts/contexts/constraints.md` (or similarly named) files. Those rules shape what the LLM *tries* to do. The guardrail layer here is the *external* check that runs around the agent and catches things the prompt alone cannot reliably guarantee.

This separation matters: guardrails are reusable across any AKD agent (and across agents you build yourself), and they can be swapped, composed, or tuned without touching the agent's prompt.

## Why this matters

Scientific assistants amplify two distinct risk classes, and they need different defenses:

- **General AI safety risks** — prompt injection, harmful requests, biased or unsafe content. These are *content-shaped* and largely domain-independent; they show up in any LLM-driven product.
- **Scientific-output risks** — hallucinated citations, ungrounded numerical claims, attribution gaps, overgeneralization, outdated confidence, multidisciplinary blind spots. These are *epistemically shaped* and only legible if the judge understands how scientific claims are normally substantiated.

A single content-moderation model is good at the first class and weak at the second. A pure LLM-judge is strong at the second but expensive and unnecessary for clearly harmful content. AKD's guardrail layer treats both as first-class concerns by shipping two complementary providers and letting them compose. The composition pattern is itself a defense-in-depth design: a fast, cheap filter rejects obvious cases up front, and a slower, domain-aware judge runs only on what survives.

This is the same architectural idea behind public guardrail systems like Llama Guard (Inan et al. 2023), Granite Guardian (Padhi et al. 2024), and NeMo Guardrails (Rebedea et al. 2023), and aligns with IBM's published practice for AI-agent compliance under the IBM AI Risk Atlas.

## How it works at a high level

Conceptually, every guardrail does the same thing:

> Take some content (an input prompt, or an agent's output) plus optional context, and return a structured verdict — pass / fail, plus the specific risks detected.

That uniform shape is what makes guardrails interchangeable. Three things follow from it:

1. **Input vs. output guarding.** The same provider can be wired to inspect what enters the agent or what leaves it. Input-side guards block harmful prompts before any tokens are generated; output-side guards catch unsafe or low-quality responses before they're shown.
2. **Composition.** Because every guardrail produces the same verdict shape, multiple guardrails can be combined: run them all and require everyone to pass, run them in parallel and accept if any passes, or run them sequentially and stop at the first failure ("fail-fast"). The third pattern is the workhorse for cost-sensitive deployments — a cheap filter ahead of an expensive judge.
3. **Result attached to the response.** When the agent answers, the guardrail verdicts are attached to the response object. Callers can read them and decide what to do — surface a warning, block delivery, route to human review, or log for evaluation. The guardrail itself does not silently swallow the response; it reports.

These ideas are implemented as decorators and runtime wrappers in `akd-core`; the API surface is summarized at the bottom of this file.

## How it's used

A typical AKD deployment uses guardrails in three ways:

- **Per-agent attachment.** A guardrail is attached to an agent at definition time (decorator) or at instantiation (runtime wrapper). The wrapped agent now produces responses carrying guardrail verdicts.
- **Provider choice.** The team picks a provider — or a composition — based on the agent's risk profile. Content-moderation-only for low-stakes interactive tools; LLM-judge for science-output agents; both, fail-fast, for production scientific assistants.
- **Per-domain tuning.** Each agent tunes the *categories* the judge cares about. A code-search agent emphasizes verification and hallucination categories; a CMR agent emphasizes consistency and attribution; a generation agent might emphasize overgeneralization and positivity bias.

The result is a single, uniform safety surface across the agent suite, with knobs that match the science of each agent's task rather than a one-size-fits-all filter.

## The two providers

AKD ships two complementary guardrail providers. They are designed to be used together, not in competition:

- **[Granite Guardian](./granite/)** — IBM Granite Guardian content moderation. Fast, opinionated, content-focused. Covers jailbreak, violence, sexual content, profanity, social bias, unethical behavior, harm, plus RAG-specific checks (groundedness, relevance, answer relevance). Runs locally via Ollama.
- **[Risk Agent](./risk-agent/)** — LLM-judge that evaluates outputs against the IBM Risk Atlas and a NASA-IMPACT Science Literature Risk taxonomy. Slower but domain-aware; detects hallucination, attribution gaps, consistency issues, overgeneralization, outdated confidence, and multidisciplinary failures. Uses a DAG-based, importance-weighted evaluation graph.

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

## References

**Guardrail systems and content moderation**

- Inan, H., et al. (2023). *Llama Guard: LLM-based Input-Output Safeguard for Human-AI Conversations.* arXiv:2312.06674. [arxiv.org/abs/2312.06674](https://arxiv.org/abs/2312.06674)
- Padhi, I., et al. (2024). *Granite Guardian.* arXiv:2412.07724. [arxiv.org/abs/2412.07724](https://arxiv.org/abs/2412.07724) — IBM's Granite Guardian model family that powers the Granite provider.
- Rebedea, T., et al. (2023). *NeMo Guardrails: A Toolkit for Controllable and Safe LLM Applications with Programmable Rails.* EMNLP System Demonstrations. [arxiv.org/abs/2310.10501](https://arxiv.org/abs/2310.10501)

**Factuality and science-output evaluation**

- Marinescu, R., et al. (2025). *FactReasoner: A Probabilistic Approach to Long-Form Factuality Assessment for Large Language Models.* arXiv:2502.18573. [arxiv.org/abs/2502.18573](https://arxiv.org/abs/2502.18573) — motivates evaluating long-form scientific outputs against retrieved evidence rather than a single overall judgement.

**AI risk taxonomies and agent compliance**

- IBM watsonx.governance — *AI agent compliance with watsonx.governance.* [ibm.com/docs/en/watsonx/saas?topic=atlas-ai-agent-compliance](https://www.ibm.com/docs/en/watsonx/saas?topic=atlas-ai-agent-compliance) — the enterprise framing this project's Risk Atlas integration follows.
- IBM AI Risk Atlas. [ibm.com/docs/en/watsonx/saas?topic=overview-ai-risk-atlas](https://www.ibm.com/docs/en/watsonx/saas?topic=overview-ai-risk-atlas) — source of the `AtlasRiskCategory` taxonomy used by the Risk Agent.

## Implementation reference

A compact summary of the developer-facing API. Reach for these when wiring guardrails into code; sub-provider docs go deeper.

**Protocol.** A guardrail implements `GuardrailProtocol`: given a `GuardrailInput` (content + optional context), produce a `GuardrailOutput` describing detected risks. Both sync `check()` and async `acheck()` are supported.

**Applying guardrails to an agent.**

- Decorator: `@guardrail(input_guardrail=..., output_guardrail=...)` on an agent class.
- Runtime wrapper: `apply_guardrails(agent, input_guardrail=..., output_guardrail=...)` on an instance.

Either way, the agent's response gets `input_guardrail_result` and `output_guardrail_result` attached, plus a computed `guardrails_passed` flag for short-circuiting.

**Composition operators.**

- `g1 & g2` — **AND**: both must pass. Parallel; fails if any detects risk.
- `g1 | g2` — **OR**: at least one must pass. Parallel; reports risks only if all fail.
- `g1 >> g2` — **fail-fast**: sequential; returns immediately on first failure. Use to put a cheap filter ahead of an expensive judge.

These produce a `CompositeGuardrail` under the hood (modes `ALL`, `ANY`, `FAIL_FAST`).

**Configuration.** Global defaults come from `akd.configs.project.GuardrailSettings` (environment-driven, e.g., `GUARDRAILS__FAIL_ON_INPUT_RISK`). Per-agent overrides go through the decorator arguments.

## Further reading

- [`granite/README.md`](./granite/README.md), [`granite/risk-categories.md`](./granite/risk-categories.md), [`granite/harm-categories.md`](./granite/harm-categories.md), [`granite/usage.md`](./granite/usage.md).
- [`risk-agent/README.md`](./risk-agent/README.md), [`risk-agent/evaluation-strategy.md`](./risk-agent/evaluation-strategy.md), [`risk-agent/risk-atlas.yaml`](./risk-agent/risk-atlas.yaml), [`risk-agent/science-literature-risks.yaml`](./risk-agent/science-literature-risks.yaml).
