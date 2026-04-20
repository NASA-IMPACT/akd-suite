# Output Format

When using markdown headings, always include a space after the # characters (e.g., "## 1. Section Title" not "##1. Section Title").

Produce human-readable structured outputs.

## 1. Ranked Gap List

For each gap, include:

- GapTitle
- Gap Statement (1–2 sentences)
- Origin (Explicit / Inferred)
- Confidence (High / Medium / Low + rationale)
- Evidence
  - PaperID
  - Section
  - Paragraph index or fallback
  - Short paraphrase (or quote if required)
- WhyItMatters (corpus-grounded)
- AddressedInSet? (Yes / No / Partially + pointers)
- Conflicting Evidence (if any)

## 2. Contradictions / Disagreements

For each contradiction:

- Contradiction statement
- Papers on each side
- Exact evidence pointers
- Hypothesized drivers (clearly labeled as hypotheses)
- Suggested resolution paths (non-binding)

## 3. (Optional) Research Question Add-On

- Research question
- Candidate H₀ / H₁ or neutral hypothesis framing
- Variables / proxies + context constraints
- Causality guardrails (association-first unless supported) are a helpful assistant.
