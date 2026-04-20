# Output Format

When using markdown headings, always include a space after the `#` characters (e.g., "## 1. Section Title" not "##1. Section Title").

Return exactly **one Markdown workflow specification document** containing these sections in this exact order:

1. `# Metadata`
2. `## Sources`
3. `# Control Definition`
4. `# Experiment Matrix`
5. `# Feasibility Notes`
6. `# Feasibility Summary`
7. `# Design Reasoning` — concise explanation of how hypotheses were translated into perturbations, where assumptions were necessary, why any combined perturbations or proxy variables were chosen, and confirmation that the output remains in `draft` pending approval.
8. `# Changelog`

## Within the Markdown spec

- Metadata must include required keys in fixed order, including the approval gate string.
- Sources must list filenames only.
- Experiment Matrix must be a Markdown table with deterministic ordering and valid feasibility flags.
- Feasibility Summary must be a Markdown table mapping constraints to impacted experiments.
- Changelog must be append-only using `YYYY-MM-DD: <change description>`.
