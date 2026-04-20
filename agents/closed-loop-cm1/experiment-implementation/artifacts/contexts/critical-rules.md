# Critical Rules

## 1. Implement, don't redesign

- Follow the Stage 3 spec exactly. Do NOT add experiments, remove experiments, or change the scientific intent.
- Preserve experiment IDs from Stage 3.

## 2. Express ALL changes as FileEdit objects

Every modification — namelist parameter changes, sounding profile edits, or file replacements — must be expressed as a `FileEdit`.

- **`edit_type="namelist_param"`**: Change a single key in a `&paramN` group.
  - Set `namelist_group` to the group name **without** the `&` (e.g. `"param9"`).
  - Set `parameter` to the key name (e.g. `"output_cape"`).
  - Set `value` to the new value (use integer for integer params, float for float).
  - Use the **CM1 reference documentation** to identify parameter names and their groups. Do NOT invent parameter names.

- **`edit_type="sounding_profile"`**: Modify a column of the `input_sounding` across a height range.
  - Set `variable` to the column: `"theta"`, `"qv"`, `"u"`, or `"v"`.
  - Set `operation`: `"add"`, `"subtract"`, `"multiply"`, or `"set"`.
  - Set `magnitude`: the numerical amount.
  - Set `z_min` / `z_max`: height bounds in metres.
  - Set `profile`: how magnitude varies across the range:
    - `"linear_ramp"`: zero delta at `z_min`, full delta at `z_max`. Formula: `delta = magnitude × (z - z_min) / (z_max - z_min)`
    - `"constant"`: uniform delta across the range.
    - `"gaussian"`: bell curve centred at midpoint of range.

- **`edit_type="file_replace"`**: Replace the entire file content.
  - Set `target_file` to the filename.
  - Use this for research questions that need a completely different sounding or any custom file.

## 3. Baseline experiments may have edits

The baseline is NOT necessarily "no changes". If the Stage 3 spec says the baseline enables diagnostics (e.g. `output_cape=1`), include those as `FileEdit` objects.

## 4. Perturbation experiments inherit baseline edits

Perturbation experiments should include all baseline edits PLUS their own additional changes.

## 5. Sounding format reference

The CM1 `input_sounding` format is:

- **Line 1** (surface): `surface_pressure(mb)  surface_theta(K)  surface_qv(g/kg)`
- **Lines 2+** (levels): `height(m)  theta(K)  qv(g/kg)  u(m/s)  v(m/s)`

When `z_min > 0`, the surface line is left unchanged. When `z_min = 0`, the surface theta or qv may be affected depending on the column.

## 6. Value types

- Fortran namelists distinguish integers from floats. If the template has `output_cape = 0,` (integer), set `value` to `1` (int), not `1.0`.
- For Fortran booleans, use `".true."` or `".false."` as strings.

## 7. Use exact parameter names

All parameter names must come from the CM1 reference documentation. Do not invent or guess parameter names. Cite evidence as file paths only (e.g. `run/config_files/hurricane_axisymmetric/namelist.input`), no quotes or excerpts.

## 8. Workspace name

Suggest a descriptive workspace directory name based on the experiment tag from the Stage 3 spec (e.g. `"cm1_stability_experiments"`).

## 9. Base template

Include `base_template` — the CM1 case template directory name from the Stage 3 spec (e.g. `"hurricane_axisymmetric"`, `"supercell"`). This is a single top-level field (same for all experiments). The Python engine uses it to fetch the correct template files. Extract it from the Stage 3 spec's control definition or experiment matrix.

## 10. Report

Produce a markdown implementation report summarising:

- Total experiments created
- Per-experiment change summary
- Any warnings or notes
- What the user should review before submitting jobs
