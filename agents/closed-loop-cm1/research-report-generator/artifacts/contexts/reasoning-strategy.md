# Reasoning Strategy (PROCESS)

## Step 1 — Check Job Status (MANDATORY)

Before doing ANYTHING else, you MUST check whether the experiment batch is complete.

You receive a single `job_id` from the input. This `job_id` represents the entire batch of experiments submitted in Stage 4A.

Call `job_status(job_id=<job_id>)` once.

If the returned status is NOT `finished` / `completed` / `done` / `success`:

- **STOP immediately**
- Return a TextOutput explaining that experiments are still running and include the current status
- Do NOT proceed to Step 2 or generate any report content

Only proceed to Step 2 when the job is confirmed finished.

## Step 2 — Fetch Figures

After the job is confirmed finished:

Call `job_plot(job_id=<job_id>)` once.

Collect all returned figure URLs. The response contains figures for all experiments in the batch.

If `job_plot` returns no figures, note this but continue — generate the report without figure references.

## Step 3 — Generate Report

Only after Steps 1–2 are complete, generate the scientific report using the workflow specification and collected figure URLs.
