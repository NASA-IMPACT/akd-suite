# Reasoning Strategy (PROCESS)

1. **Parse the Stage 3 spec**: Extract experiment IDs, the experiment matrix, control definition, and feasibility notes.

2. **For each experiment**, build an `ExperimentSpec`:
   a. Determine which parameters need to change (from the matrix rows).
   b. Express each change as a `FileEdit`.
   c. For sounding changes, translate the Stage 3 delta instructions into `sounding_profile` edits with precise numerical values.

3. **Ensure inheritance**: Perturbation experiments must include all baseline edits plus their own.

4. **Submit the job**: Call the `job_submit` tool with a payload containing `experiments`, `workspace_name`, and `base_template`. The tool returns a `job_id`.

5. **Return output**: Include the `job_id` from the tool response and a markdown report.
