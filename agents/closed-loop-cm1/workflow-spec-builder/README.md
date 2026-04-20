# Workflow Spec Builder (CM1 Stage 3)

The **Stage-3 Workflow Spec Builder** designs scientifically traceable, feasibility-aware simulation experiments and documents them as a single execution-ready Markdown workflow specification — for either **CM1** or **WRF**, never both in the same document.

## At a glance

| | |
| --- | --- |
| **Stage** | 3 (in the CM1 closed-loop pipeline) |
| **Purpose** | Convert research questions into a complete Markdown experiment spec with baseline + perturbations, parameter sweeps, namelist / input_sounding deltas, and feasibility notes |
| **Input** | `stage_1_hypotheses` (Gap Agent output) + `stage_2_feasibility` (Capability & Feasibility Mapper output) |
| **Output** | `spec` (full Markdown workflow specification, always `status: draft`) + `reasoning` (structured design reasoning) |
| **Tools** | None wired; embeds `cm1_readme.md` as instruction context |
| **Source** | [`akd_ext/agents/closed_loop_cm1/workflow_spec_builder.py`](https://github.com/NASA-IMPACT/akd-ext) |

## What a successful run produces

A single Markdown workflow specification with this exact section order:

1. `# Metadata`
2. `## Sources`
3. `# Control Definition`
4. `# Experiment Matrix`
5. `# Feasibility Notes`
6. `# Feasibility Summary`
7. `# Design Reasoning`
8. `# Changelog`

The spec always ships with `status: draft`. Never self-upgrades to `approved`. Human approval is required before Stage 4 can act on it.

## What it never does

- Runs simulations
- Creates directories
- Edits model files directly
- Mixes CM1 and WRF experiments in one spec
- Self-upgrades to `approved`
- Silently drops problematic experiments
- Uses placeholders like `null`, `TBD`, or `N/A`

## Deeper reading

- [`artifacts/`](./artifacts/) — full knowledge layer.
- Upstream: [`akd_ext/agents/closed_loop_cm1/workflow_spec_builder.py`](https://github.com/NASA-IMPACT/akd-ext).
