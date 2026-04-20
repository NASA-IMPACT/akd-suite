# Research Report Generator (CM1 Stage 5)

The **Stage-5 Research Report Generator** produces a publication-style scientific report from a completed CM1 experiment batch. It verifies job completion, retrieves result figure URLs, and drafts a Markdown report in the style of a peer-reviewed atmospheric-science journal article.

## At a glance

| | |
| --- | --- |
| **Stage** | 5 (in the CM1 closed-loop pipeline) |
| **Purpose** | After Stage-4's job completes, verify completion, fetch figure URLs, and generate a publication-style scientific report |
| **Input** | `workflow_spec` (Stage-3 markdown — primary source of scientific context) + `job_id` (from Stage-4) |
| **Output** | `report` — complete Markdown scientific report (Abstract, Introduction, Model and Methodology, Results, Discussion, Conclusion, Figures) |
| **Tools** | `job_status`, `job_plot` via `Job_Management_Server` hosted MCP |
| **Source** | [`akd_ext/agents/closed_loop_cm1/research_report_generator.py`](https://github.com/NASA-IMPACT/akd-ext) |

## What a successful run produces

A complete Markdown scientific report following the journal-article structure, with every `job_plot`-returned figure URL embedded via markdown image syntax and every interpretation tagged "(pending researcher validation)". The report carries the disclaimer *"This report was generated with AI assistance and requires researcher validation before publication."*

## What it never does

- Generates report content before confirming `job_status` returns a finished state
- Invents quantitative numbers (it has figures, not raw metrics)
- Designs new experiments or suggests parameter changes beyond what the Stage-3 feasibility notes identified
- Fabricates numerical comparisons
- Includes code or technical implementation details
- Reproduces the full experiment matrix table

## Deeper reading

- [`artifacts/`](./artifacts/) — full knowledge layer.
- Upstream: [`akd_ext/agents/closed_loop_cm1/research_report_generator.py`](https://github.com/NASA-IMPACT/akd-ext).
