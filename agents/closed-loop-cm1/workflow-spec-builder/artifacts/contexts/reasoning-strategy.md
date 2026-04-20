# Reasoning Strategy (PROCESS)

Follow this sequence every time:

## 1. Ingest and normalize inputs

- Extract research-question tags/IDs and hypotheses from Stage-1.
- Determine whether the requested document is CM1 or WRF only.

## 2. Define baseline

- Use the baseline/control already provided by inputs or user direction.
- Do not autonomously replace the user's baseline choice.
- Create baseline ID using the established naming convention.

## 3. Generate candidate perturbations

- Map each hypothesis to one or more perturbations.
- Express perturbations as `namelist` deltas and/or `input_sounding` deltas.
- Prefer clear, non-redundant experiments that directly test the hypotheses.

## 4. Apply feasibility review

- Preserve hard constraints explicitly in notes.
- Example: if independent Cd/Ce control is required, maintain constraints such as `cecd=1`, `sfcmodel=1`, and `ipbl ∈ {0,2}`, and note that certain `ipbl` values break independence or require code change.

## 5. Resolve conflicts and redundancy

- Remove duplicates.
- Collapse overlapping experiments when they test the same mechanism.
- If an unsupported request appears, propose the nearest feasible proxy and flag it.

## 6. Build the Markdown spec

Use the exact required section order:

`# Metadata` → `## Sources` → `# Control Definition` → `# Experiment Matrix` → `# Feasibility Notes` → `# Feasibility Summary` → `# Changelog`.

## 7. Populate the Experiment Matrix

- Use a Markdown table.
- Use **one row per parameter change**, not one row per experiment.
- Include required columns in the required order.
- Use inline semicolon-separated deltas, alphabetized within each cell.
- Include traceability fields such as `rq_tag_or_rq_id`, `hypothesis_id` when available, what the row tests, and feasibility constraints.

## 8. Summarize feasibility

- Add a narrative `# Feasibility Notes` section describing important constraints, blockers, assumptions, and mitigation logic.
- Add a `# Feasibility Summary` Markdown table mapping `constraint` to comma-separated, lexically sorted impacted experiments.

## 9. Stop at draft

- End after producing the complete draft spec.
- Ask for approval rather than continuing to approval state automatically.
