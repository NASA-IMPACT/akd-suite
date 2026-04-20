# Labs — Benchmarking

How benchmark runs work in Labs, and how to use them to compare agents objectively.

## The model

A **benchmark run** pairs:

- An **agent** (by id, with a pinned config snapshot — system prompt, model, tools) and
- A **suite** of test prompts.

The worker executes each prompt through the agent, captures a **result** per prompt (including a grade), and produces an aggregate view.

## Suites

A suite is a curated set of test prompts representing the agent's intended workload. Good suites cover:

- **Happy path** — typical queries the agent should handle well.
- **Edge cases** — ambiguous, underspecified, or adversarial inputs.
- **Off-scope** — queries the agent should refuse or redirect.
- **Regression traps** — queries that failed previously (add them to prevent recurrence).

Each prompt in the suite carries expected behavior metadata (what's an acceptable response, what counts as failure).

## Grading

Results are graded per the suite's criteria. Labs supports grading approaches like:

- **Rubric-based** — an LLM judge applies a rubric to the agent's response.
- **Exact-match** — compare structured output fields to expected values.
- **Containment / substring** — check that the response contains expected phrases.
- **Manual** — leave the grade open for a human reviewer.

Specific grading implementations live in `services/` in the upstream repo; consult there for the exact set and how to add new ones.

## Cost tracking

Every result includes the token counts and estimated cost (per Labs's pricing table for the model). Aggregating across a run gives a total cost — useful for projecting production spend and for comparing the cost-efficiency of different agent configurations.

## Comparing runs

The UI supports side-by-side comparison of two runs — handy for:

- Before-and-after prompt changes.
- Different models on the same suite.
- Same agent on different suites.

## Exporting

Run results can be exported as CSV, JSON, or HTML via `api/export.py`. Useful for sharing results with stakeholders or for downstream analysis in notebooks.

## Relationship to traces

Each benchmark result is backed by a trace — so when a grader says "failed," the full trace of that run (tool calls, reasoning, tokens) is one click away for debugging.

## Tips

- Keep suites small and focused. Large suites are expensive and slow.
- Include diversity across input types rather than many near-duplicates.
- Track which prompts are the most sensitive to prompt changes — those are your signal prompts.
- Re-run the same suite before and after every meaningful config change; treat the diff as part of code review for prompt engineering.

## Where to read next

- [`user-guide.md`](./user-guide.md) — running benchmarks in the UI.
- [`agent-development-lifecycle.md`](./agent-development-lifecycle.md) — where benchmarking fits in the loop.
- [`architecture.md`](./architecture.md) — the data model for Run / Result / TraceLog.
