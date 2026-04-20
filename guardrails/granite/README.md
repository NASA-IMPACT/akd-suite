# Granite Guardian

AKD's **Granite Guardian** provider wraps IBM's [Granite Guardian](https://huggingface.co/ibm-granite) content-moderation family as an AKD guardrail. It runs locally via Ollama and comes in two modes — single-risk and multi-risk — with complementary trade-offs.

## What it is

A framework-level AKD guardrail provider that classifies content as harmful / safe across a fixed risk taxonomy. Implemented in `akd-core` under `akd.guardrails.providers.granite_guardian` and exposed through the same `GuardrailProtocol` any other guardrail uses.

## How it works

Granite Guardian ships in two AKD-exposed forms:

### Single-risk mode (`GraniteGuardianTool`)

- Backed by `granite3-guardian:2b` or `granite3-guardian:8b` (or `granite3.3-guardian:8b` for chain-of-thought reasoning).
- Checks **one risk category at a time**.
- To evaluate multiple categories simultaneously, AKD runs those calls in parallel with a configurable concurrency (default 3 in-flight).
- Models are reasoning-capable; the 3.3-8B variant supports "thinking" mode that surfaces its chain of reasoning.

### Multi-risk mode (`MultiRiskGraniteGuardianTool`)

- Backed by `granite-guardian-3.2-5b-multi-harm-GGUF`, a model that detects multiple harms in a single pass.
- **Two-step inference**:
  - Step 1: Determine whether the content is harmful (yes/no with confidence).
  - Step 2: If yes, extract specific harm categories with per-category confidence scores.
- A configurable `score_threshold` filters low-confidence detections.

## Ollama dependency

Both variants require a running Ollama instance reachable at `OLLAMA_BASE_URL` (default `http://localhost:11434`). There is no documented fallback for an unreachable Ollama — calls will time out or fail. In production AKD Flow deployments, Ollama is part of the stack (see [`../../flow/deployment.md`](../../flow/deployment.md)).

## When to use which mode

| Situation | Prefer |
| --- | --- |
| Low-cost, high-volume input filtering with a narrow category set | Single-risk, 2B model |
| Higher-quality single-category judgement with reasoning summary | Single-risk, 3.3-8B (thinking mode) |
| Broad content moderation across many categories in one pass | Multi-risk |
| Budget-sensitive multi-category evaluation | Multi-risk with tuned `score_threshold` |

## How it composes

Like every AKD guardrail, Granite Guardian plugs into the composition operators defined at [`../README.md`](../README.md):

- `granite & other` — both must pass (parallel AND).
- `granite | other` — at least one passes (parallel OR).
- `granite >> other` — run Granite first, only invoke `other` if Granite passes (fail-fast). Useful to put a fast filter ahead of an expensive judge.

## Companion docs

- [`risk-categories.md`](./risk-categories.md) — the input-mode category taxonomy (`GraniteRiskCategory`).
- [`harm-categories.md`](./harm-categories.md) — the output-mode harm taxonomy (`GraniteHarmCategory`) returned by the multi-risk model.
- [`usage.md`](./usage.md) — how to apply Granite Guardian to an agent (via `@guardrail`, `apply_guardrails`, and composition) and how to tune thresholds.

## Upstream

Implementation: `akd.guardrails.providers.granite_guardian` in [NASA-IMPACT/accelerated-discovery](https://github.com/NASA-IMPACT/accelerated-discovery). The category enums live under `akd.guardrails.categories.granite`.
