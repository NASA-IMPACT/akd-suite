# Experiment Implementation (CM1 Stage 4)

The **Stage-4A Experiment Implementation Planner** translates Stage-3 workflow specs into structured `FileEdit` JSON and submits the experiment batch as a job via an MCP tool call. It does not create files or execute simulations itself — it produces structured output that a deterministic Python engine executes.

## At a glance

| | |
| --- | --- |
| **Stage** | 4 (in the CM1 closed-loop pipeline) |
| **Purpose** | Convert Stage-3 draft spec → structured `ExperimentSpec` / `FileEdit` JSON → submit via `job_submit` MCP tool |
| **Input** | `stage_3_spec` — Stage-3 workflow specification markdown |
| **Output** | `job_id` (from `job_submit`) + `report` (markdown implementation summary) |
| **Tools** | `job_submit` via `Job_Management_Server` hosted MCP; embeds `cm1_readme.md` as instruction context |
| **Source** | [`akd_ext/agents/closed_loop_cm1/experiment_implementation.py`](https://github.com/NASA-IMPACT/akd-ext) |

## What a successful run produces

- A list of `ExperimentSpec` objects (one per experiment) containing `FileEdit` lists that describe every modification to the template files.
- A call to `job_submit` with the full payload (`experiments`, `workspace_name`, `base_template`).
- A `job_id` returned by the MCP tool and a markdown implementation report.

The Python engine downstream of this agent copies template files into each experiment directory, applies the `FileEdit` list deterministically, and generates SLURM scripts and READMEs.

## What it never does

- Creates files or runs commands itself
- Adds or removes experiments from the Stage-3 spec
- Changes scientific intent
- Invents parameter names
- Executes the submitted job (that's the Python engine's responsibility)

## Deeper reading

- [`artifacts/`](./artifacts/) — full knowledge layer including the `FileEdit` / `ExperimentSpec` data model.
- Upstream: [`akd_ext/agents/closed_loop_cm1/experiment_implementation.py`](https://github.com/NASA-IMPACT/akd-ext).
