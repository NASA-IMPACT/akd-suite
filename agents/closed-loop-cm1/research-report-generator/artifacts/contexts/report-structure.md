# Report Structure

The report MUST contain these sections in this exact order:

## 1. Abstract

- 3–5 sentences summarising the research question, experimental method, key result, and scientific implication.

## 2. Introduction

- State the scientific question and its importance in atmospheric science.
- Describe relevant background (what is known about the topic from the workflow spec's feasibility notes and evidence).
- State the hypothesis being tested (from the workflow spec).

## 3. Model and Methodology

- Describe the CM1 model setup from the workflow spec's Control Definition:
  - Configuration (axisymmetric vs 3D, grid resolution, integration time)
  - Baseline template used
  - What was held fixed (surface fluxes, drag, physics schemes)
- Describe the experiment design from the Experiment Matrix:
  - Number of experiments (baseline + perturbations)
  - What parameter was varied and the specific values/modifications
  - What diagnostics were enabled
- Reference the causality guardrails from the workflow spec.

## 4. Results

- Describe what the figures show.
- Reference each figure by its filename from the URL.
- Compare experiments qualitatively based on what the experiment matrix says each one tests (e.g., "The stable perturbation experiment was designed to test whether increased stability suppresses convection").
- Note the expected outcomes from the workflow spec's `what_this_tests` column and describe whether the figures appear consistent with those expectations.
- Flag any results as "(pending quantitative validation by researcher)".

## 5. Discussion

- Interpret results in context of the hypothesis from the workflow spec.
- Discuss the physical mechanisms implied by the experiment design.
- Note caveats and limitations from the workflow spec's Feasibility Notes (e.g., axisymmetric limitations, moisture/RH coupling effects, CONSTRAINT_DEPENDENT items).
- Reference any interpretation risks noted in the workflow spec.

## 6. Conclusion

- Restate whether the hypothesis appears supported based on available figures.
- Summarise what the experiment design tested.
- Suggest next steps or extensions based on the workflow spec's feasibility summary and any unresolved constraints.

## 7. Figures

- List all figures with descriptive captions derived from the experiment design context.
- Embed each figure using markdown image syntax: `![Caption](url)`
- Use the exact URLs returned by `job_plot`.
- Every figure URL collected MUST appear in the report as an embedded image.
