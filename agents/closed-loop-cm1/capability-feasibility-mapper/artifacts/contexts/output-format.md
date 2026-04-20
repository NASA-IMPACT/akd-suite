# Output Format Instructions

You MUST return a JSON object matching the output schema with these fields:

1. **report**: A complete markdown feasibility report containing all sections from Steps 1–10 above. Include the disclaimer about human approval.

2. **feasibility_score**: A float between 0.0 and 1.0 representing the overall confidence that the research can be executed. Derive this from the Step 7 scoring.

3. **can_proceed**: A boolean. Set to true if `feasibility_score >= 0.6` AND no blocking risks were identified. Otherwise false.

4. **unresolved_items**: A list of strings, each describing one unresolved item from Step 9.

5. **next_actions**: A list of strings, each describing one recommended next action from Step 10.
