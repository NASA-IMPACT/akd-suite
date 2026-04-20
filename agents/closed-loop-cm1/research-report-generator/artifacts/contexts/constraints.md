# Constraints

## Scientific integrity

- Do NOT invent quantitative numbers. You have figures but not raw metrics. Describe trends and comparisons qualitatively.
- All interpretations MUST include "(pending researcher validation)".
- Include the disclaimer: *"This report was generated with AI assistance and requires researcher validation before publication."*

## What you CAN extract from the workflow spec

- Research question and hypothesis text
- Experiment names and what each tests
- Parameter values and modifications (from experiment matrix `delta_instructions`)
- Fixed parameters and guardrails
- Feasibility constraints and risks
- Expected signals if hypothesis holds

## Style

- Use passive voice where conventional in scientific writing.
- Be specific about experiment design details from the workflow spec.
- Use SI units throughout.
- Reference figures using markdown image syntax: `![Caption](url)`
- When using markdown headings, always include a space after the `#` characters (e.g., "## 1. Section Title" not "##1. Section Title").

## What NOT to do

- Do NOT design new experiments or suggest parameter changes beyond what the workflow spec's feasibility notes already identify.
- Do NOT fabricate numbers or quantitative comparisons.
- Do NOT include code or technical implementation details.
- Do NOT include file paths to source code or config files.
- Do NOT reproduce the full experiment matrix table — summarise it narratively.
