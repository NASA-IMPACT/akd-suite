# Granite Guardian — Usage

How to apply Granite Guardian to an agent, tune it, and compose it with other guardrails.

## Applying Granite Guardian to an agent

Granite Guardian is a standard `GuardrailProtocol` implementation. It plugs into an agent through the two universal entry points documented in [`../README.md`](../README.md):

- As a class-level decorator: `@guardrail(input_guardrail=<granite instance>, output_guardrail=...)` on the agent class.
- As a runtime wrapper: `apply_guardrails(agent_instance, input_guardrail=<granite instance>, output_guardrail=...)`.

Both wrap the agent's response in a mixin that carries `input_guardrail_result` / `output_guardrail_result` / `guardrails_passed`.

## Modes

### Single-risk mode (`GraniteGuardianTool`)

Best when you want to check one or a few categories with the smaller/cheaper Granite Guardian models. You pick which categories to check and AKD dispatches per-category calls to Ollama in parallel (default concurrency: 3).

Configurable parameters:

- `risk_categories` — list of `GraniteRiskCategory` enum members (see [`risk-categories.md`](./risk-categories.md)).
- Model selection — defaults to 8B; the 2B model is cheaper/faster; the 3.3-8B variant supports a "thinking" mode for chain-of-thought output.
- Ollama connection — `ollama_base_url`, `ollama_type` (`chat` or `server`), `timeout`.

### Multi-risk mode (`MultiRiskGraniteGuardianTool`)

Best when you want broad coverage across many categories in a single call. The model returns a Title Case harm list (see [`harm-categories.md`](./harm-categories.md)) which AKD maps back to the input taxonomy via the `maps_to` field.

Configurable parameters:

- `score_threshold` (0.0–1.0) — filter out detections below this confidence. Default is 0.0 (no filtering). Guidance: tune empirically against your content distribution; there is no universal recommended value.
- Standard Ollama connection parameters.

## Composition

Granite Guardian implements the composition operators available to every AKD guardrail:

- `granite & other` — parallel AND: both must pass.
- `granite | other` — parallel OR: at least one passes.
- `granite >> other` — fail-fast: run Granite first; if it fails, don't call `other`.

A common pattern is `granite >> risk_agent` — fast Granite filter first, followed by the more expensive Risk Agent only when Granite gives the green light. See [`../risk-agent/`](../risk-agent/) for the Risk Agent provider.

## Configuration defaults

Global defaults come from `akd.configs.project.GuardrailSettings` (environment-driven — e.g., `GUARDRAILS__FAIL_ON_INPUT_RISK`, `GUARDRAILS__FAIL_ON_OUTPUT_RISK`). Per-agent overrides flow through the decorator or wrapper arguments.

## Practical notes

- **Ollama is required.** Ensure Ollama is reachable at `OLLAMA_BASE_URL` before enabling Granite guardrails. AKD Flow bundles Ollama in its Docker stack; standalone deployments need to provide it.
- **Category choice matters.** Picking a broad category like `harm` plus a specific one like `social_bias` is fine; Granite will check each category independently.
- **RAG categories** (`groundedness`, `relevance`, `answer_relevance`) are particularly valuable for retrieval-augmented agents where "safe but ungrounded" is a real failure mode.
- **Thinking mode** (3.3-8B) emits a reasoning summary in addition to the classification. Costs more tokens; useful for debugging or audit.
