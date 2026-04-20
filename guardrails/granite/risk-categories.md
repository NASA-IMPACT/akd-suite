# Granite Risk Categories

The `GraniteRiskCategory` enum defines the **input-side** risk taxonomy that Granite Guardian single-risk mode checks. Each category is a snake-case identifier plus metadata (description, severity).

Values mirrored verbatim from `akd.guardrails.categories.granite.GraniteRiskCategory`.

## Core harm categories

| Identifier | Description | Severity |
| --- | --- | --- |
| `harm` | General harmful content | high |
| `social_bias` | Socially biased content | normal |
| `profanity` | Profane language | normal |
| `sexual_content` | Sexual content | high |
| `unethical_behavior` | Unethical behavior | normal |
| `violence` | Violence-related content | high |
| `jailbreak` | Jailbreak attempts | high |

## RAG categories

| Identifier | Description | Severity |
| --- | --- | --- |
| `groundedness` | Response grounded in context | normal |
| `relevance` | Content relevance to query | normal |
| `answer_relevance` | Answer relevance to question | normal |

## Notes

- These identifiers are supplied as inputs to the single-risk Granite Guardian tool.
- A single Granite Guardian call checks one category at a time; evaluating multiple categories means multiple parallel calls.
- RAG categories (`groundedness`, `relevance`, `answer_relevance`) are particularly useful for output-side checks in retrieval-augmented workflows.

## See also

- [`harm-categories.md`](./harm-categories.md) — the output taxonomy returned by the multi-risk model (Title Case values with a `maps_to` mapping back to this enum).
- [`README.md`](./README.md) — Granite Guardian overview.
