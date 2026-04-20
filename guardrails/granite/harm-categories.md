# Granite Harm Categories

The `GraniteHarmCategory` enum defines the **output-side** harm taxonomy returned by the Granite Guardian multi-harm model (`granite-guardian-3.2-5b-multi-harm-GGUF`). Values are Title Case strings returned comma-separated by the model; each includes a `maps_to` field pointing to the corresponding input identifier in `GraniteRiskCategory`.

Values mirrored verbatim from `akd.guardrails.categories.granite.GraniteHarmCategory`.

## Harm categories (output)

| Title Case value | Description | Severity | Maps to |
| --- | --- | --- | --- |
| `Social Bias` | Socially biased content | normal | `social_bias` |
| `Jailbreaking` | Jailbreak attempts | high | `jailbreak` |
| `Violence` | Violence-related content | high | `violence` |
| `Profanity` | Profane language | normal | `profanity` |
| `Sexual Content` | Sexual content | high | `sexual_content` |
| `Unethical Behavior` | Unethical behavior | normal | `unethical_behavior` |
| `Harmful` | General harmful content | high | `harm` |

## Non-harmful markers

The multi-harm model also returns markers for safe content so callers can distinguish "model ran, no harms" from "model failed":

| Title Case value | Description | Severity |
| --- | --- | --- |
| `Not harmful prompt` | Prompt is not harmful | low |
| `Not harmful response` | Response is not harmful | low |

## Usage notes

- The model returns Title Case strings exactly as listed above; consumers parse the comma-separated output and look up each value in this enum.
- The `maps_to` field enables consistent cross-format handling (e.g., translating a multi-risk output back to the single-risk category set used elsewhere in AKD).

## See also

- [`risk-categories.md`](./risk-categories.md) — the input-side `GraniteRiskCategory` taxonomy.
- [`README.md`](./README.md) — Granite Guardian overview.
