# Constraints and Style Rules

You must obey all of the following:

## 1. Design only; no execution

- Do not run simulations.
- Do not create directories.
- Do not edit model files directly.
- Express changes only as instructions/deltas for `namelist` and `input_sounding`.

## 2. Single deliverable

- Output exactly **one Markdown document**.
- Markdown only; no embedded JSON or YAML blocks in the final deliverable.

## 3. Single-model only

- The spec must be for **CM1** or **WRF** only.
- Never mix CM1 and WRF experiments in one spec.

## 4. Approval gate

- Always emit `status: draft` unless the user explicitly approves.
- Never self-upgrade to `approved`.

## 5. Missing information behavior

- Produce a complete draft even when some details are missing.
- Do not invent runtime, compute, or diagnostics details.
- Do not print placeholders like `null`, `TBD`, or `N/A`.
- Omit unavailable fields, and place necessary uncertainty as explicit assumptions or notes in narrative sections.

## 6. Determinism

- Use fixed section order.
- Order experiments deterministically: baseline first, then perturbations in lexical order.
- Order delta items alphabetically within each cell.
- Use stable, repeatable wording and structure for identical inputs.

## 7. Naming and labels

- Baseline experiment ID should follow `EXP_{tag}_baseline` unless an established input convention says otherwise.
- Perturbation IDs should follow `EXP_{tag}_001`, `EXP_{tag}_002`, etc.
- `control_label` must be exactly `baseline` for baseline rows and blank for all non-baseline rows.

## 8. Feasibility handling

- Do not silently drop problematic experiments.
- Keep feasible, risky, and conditional items when useful, but flag them.
- Use feasibility flags from this enum only: `OK`, `INFEASIBLE_REQUIRES_CODE_CHANGE`, `CONDITIONAL_BLOCKER`, `CONSTRAINT_DEPENDENT`.
- If multiple apply, use most-severe-wins ordering: `INFEASIBLE_REQUIRES_CODE_CHANGE` > `CONDITIONAL_BLOCKER` > `CONSTRAINT_DEPENDENT` > `OK`.
- If a requested variable or perturbation is unsupported, propose the closest feasible proxy and explain it.

## 9. Default experiment design policy

- Prefer **baseline + perturbations**.
- Allow combined perturbations when hypotheses share a causal chain.
- Default maximum is **5 experiments total** unless the user requests more.

## 10. Provenance

- Include a `## Sources` section with **filenames only**.
- No inline, row-level, or claim-level citations in the generated spec.
- Include CM1 README filename only if it was used.
