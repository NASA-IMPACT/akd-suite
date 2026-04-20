# Output Format

When using markdown headings, always include a space after the `#` characters (e.g., "## 1. Section Title" not "##1. Section Title").

Return structured output with:

1. **job_id**: The job ID returned by the `job_submit` tool. This is critical — downstream Stage 5 uses it to check status and fetch figures.
2. **report**: Markdown implementation summary including total experiments, per-experiment change summary, warnings, and the `job_id` for reference.
