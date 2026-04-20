# CM1 Pipeline — Overview

The closed-loop CM1 pipeline chains five agents through a scientific research workflow for atmospheric simulation. Each stage has a **human approval gate** — no stage transitions automatically. The pipeline ends at a publication-style report drafted from completed runs.

## The chain

```
  Corpus of papers
         │
         ▼
  ┌──────────────────┐
  │ Stage 1          │  Gap Agent
  │ Gap Identification│
  └────────┬─────────┘
           │ research questions + hypotheses (markdown)
           ▼
  ┌──────────────────┐
  │ Stage 2          │  Capability & Feasibility Mapper
  │ Feasibility Map   │  • parses RQs, extracts hypothesis requirements
  └────────┬─────────┘  • retrieves evidence from CM1 docs
           │            • computes compute feasibility (Part A from configs)
           │            • assesses cluster fit (Part B from cluster docs)
           │ feasibility report + score + can_proceed flag
           ▼
  ┌──────────────────┐
  │ Stage 3          │  Workflow Spec Builder
  │ Experiment Spec   │  • CM1 OR WRF (never both)
  └────────┬─────────┘  • baseline + perturbations
           │            • namelist / sounding deltas expressed as instructions
           │ single draft markdown spec (status: draft)
           ▼
  ┌──────────────────┐
  │ Stage 4          │  Experiment Implementation
  │ Implementation    │  • translates spec to structured FileEdit JSON
  └────────┬─────────┘  • submits experiment batch via MCP job_submit
           │ job_id + implementation report
           ▼
  ┌──────────────────┐
  │ Stage 5          │  Research Report Generator
  │ Report            │  • MCP job_status — must be finished
  └──────────────────┘  • MCP job_plot — fetches figure URLs
                       • produces publication-style scientific markdown report
```

## What crosses each hand-off

| Hand-off | What passes | Format |
| --- | --- | --- |
| Stage 1 → Stage 2 | `research_questions_md` | Gap Agent's structured markdown report (research questions + hypotheses + causality guardrails) |
| Stage 2 → Stage 3 | `stage_1_hypotheses` + `stage_2_feasibility` | Stage-1 markdown plus Stage-2 feasibility report |
| Stage 3 → Stage 4 | `stage_3_spec` | Stage-3 markdown workflow specification (experiment matrix + control definition + feasibility notes) |
| Stage 4 → Stage 5 | `workflow_spec` + `job_id` | Stage-3 spec (primary scientific source) plus the Stage-4 `job_id` identifying the submitted batch |

## Human approval gates

Every stage has explicit HITL pauses baked into its prompt:

- **Stage 1** pauses after each of its six stages (scope, extraction, gap-matrix, identification, research questions, prioritization).
- **Stage 2** carries the disclaimer "this report provides capability/feasibility assessments and evidence paths only. It does not make a final decision to run experiments; human approval is required."
- **Stage 3** stops at `status: draft` unless the user explicitly approves. Never self-upgrades to `approved`.
- **Stage 4** ends by calling `job_submit` — but the Stage-3 spec (draft) must already have been user-approved before Stage 4 runs.
- **Stage 5** halts immediately if `job_status` is not `finished` / `completed` / `done` / `success`. Only generates a report after confirmed completion.

## MCP tool usage

Stages 2 and 3 have **no MCP tools**. They are pure reasoning agents that operate over their inputs plus embedded context markdowns (`cm1_readme.md`, and for Stage 2 also `cluster_it.md`).

Stages 4 and 5 use the **`Job_Management_Server`** hosted MCP (env var `EXPERIMENT_STATUS_MCP_URL`) with different allowed tool sets:

| Stage | MCP tool | Purpose |
| --- | --- | --- |
| 4 | `job_submit` | Submit the batch of experiments and receive a `job_id` |
| 5 | `job_status` | Verify the batch is finished |
| 5 | `job_plot` | Fetch the figure URLs for the completed batch |

## Why it's "closed-loop"

Despite the linear top-to-bottom diagram, the pipeline is conceptually closed because each stage's output is evidence-traceable back to the previous stage, and because the human reviewer can reject any stage and reroute — for example, sending a Stage-2 feasibility report back to Stage 1 for a new set of research questions if the original scope proves infeasible. The CM1 pipeline is designed around human-decision branches, not straight automation.
