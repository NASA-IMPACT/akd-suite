# Data Model — FileEdit and ExperimentSpec

The agent emits structured data conforming to these schemas (defined in akd-ext).

## FileEdit

A single edit to a CM1 input file.

Fields:

- `target_file` — target filename: `namelist.input` or `input_sounding`.
- `edit_type` — one of:
  - `namelist_param` — change a single key in a `&paramN` group.
  - `sounding_profile` — modify a column of the `input_sounding` across a height range.
  - `file_replace` — replace the entire file content.

For `edit_type="namelist_param"`:

- `namelist_group` — group name without `&` prefix (e.g. `param9`).
- `parameter` — parameter key name (e.g. `output_cape`).
- `value` — new value; int, float, string, or bool as appropriate.

For `edit_type="sounding_profile"`:

- `variable` — sounding column: `theta`, `qv`, `u`, or `v`.
- `operation` — `add`, `subtract`, `multiply`, or `set`.
- `magnitude` — numerical magnitude of the change.
- `z_min` — lower height bound in metres.
- `z_max` — upper height bound in metres.
- `profile` — shape: `linear_ramp`, `constant`, or `gaussian`.

For `edit_type="file_replace"`:

- Replace-content semantics are carried through other fields as appropriate.

## ExperimentSpec

A complete specification for one experiment.

Fields:

- `experiment_id` — ID from Stage 3 spec (e.g. `EXP_stability_baseline`).
- `description` — what this experiment tests.
- `is_baseline` — whether this is the baseline.
- `feasibility_flag` — from Stage 3: `OK`, `INFEASIBLE_REQUIRES_CODE_CHANGE`, `CONDITIONAL_BLOCKER`, `CONSTRAINT_DEPENDENT`.
- `edits` — ordered list of `FileEdit` objects.

## Payload shape passed to `job_submit`

Not a separate schema in akd-ext but implied by the PROCESS: an object containing

- `experiments` — the list of `ExperimentSpec` objects.
- `workspace_name` — string (e.g. `cm1_stability_experiments`).
- `base_template` — CM1 case template directory (e.g. `hurricane_axisymmetric`).
