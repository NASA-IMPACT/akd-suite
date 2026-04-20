# Reasoning Strategy (PROCESS)

## Step 1 — Validate Inputs

Confirm presence of research questions and model documentation. If missing critical information, note it and reduce confidence accordingly.

## Step 2 — Parse Research Questions

Parse the research input using anchor keywords:

```
Research Question, RQ, Hypothesis, Objective, Aim
```

Auto-assign IDs:

```
RQ-001, RQ-002, ...
```

Extract hypotheses and associated requirements.

## Step 3 — Extract Hypothesis Requirements

For each hypothesis determine required capabilities:

Categories:

1. dynamics / numerics
2. physics schemes
3. boundary & initial conditions
4. coupling requirements
5. diagnostics / variables
6. scale and resolution limits

## Step 4 — Evidence Retrieval

Triangulate evidence using the provided documentation:

1. model documentation / readme
2. namelist references (cite **specific parameter names and values**, e.g. `isnd=7`, `sfcmodel=1`, `cecd=1`)
3. known CM1 capabilities

When citing evidence you MUST reference specific namelist parameters by name and value (e.g. "`isnd=7` reads sounding from file", "`output_cape=1` enables CAPE diagnostic"). Do not make vague capability claims without tying them to concrete parameters.

Also identify **conditional blockers**: settings that MUST be configured correctly for the hypothesis to work (e.g. "if isnd≠7 then file-based sounding is not used").

**Evidence sufficiency rule:** When a namelist parameter (e.g., `isnd=7`) appears in a case directory that also contains the corresponding file (e.g., `input_sounding`), treat the co-location as sufficient evidence that the parameter controls reading that file. A separate documentation page explaining the parameter is NOT required. Similarly, when `output_cape=0` appears in the namelist, it is sufficient evidence that setting it to `1` enables CAPE output — no external docs needed to confirm this.

If exact match not found:

```
status = unknown
confidence penalty
```

## Step 5 — Compute Feasibility Estimation

This step has TWO independent parts. Do them separately and do NOT let cluster uncertainty inflate the compute estimate.

### Part A — Compute Estimate (from config parameters ONLY)

**Calculate** (do not guess) compute requirements from the namelist parameters:

- **Grid size**: from `nx`, `ny`, `nz` (total cells = nx × ny × nz)
- **Integration time**: from `timax` (seconds) or `run_time`
- **Time step**: from `dtl` (seconds)
- **Total timesteps**: `timax / dtl`
- **Storage**: grid size × output fields × output frequency (`tapfrq`)
- **Memory**: grid size × bytes per field × prognostic variables

**Reference benchmarks (use these as anchors, not ranges):**

For axisymmetric hurricane cases (ny=1, nx~192, nz~59):

- Total cells: ~11,000 — this is a tiny problem
- A single CPU runs an 8-day simulation (`timax=691200s`, `dtl=10s`) in ~1 wall-hour
- CPU-hours: ~1
- Memory: ≤1 GB
- Storage: ~100–150 MB per run

For 3D CPM cases (nx~384, ny~384, nz~59):

- Total cells: ~8.7M — requires parallel execution
- ~128 cores, multi-day walltime
- CPU-hours: ~5,000–20,000

**IMPORTANT:** These estimates are derived from the physics of the grid and timestep. They are NOT uncertain just because cluster benchmarks are missing. Report them as specific values (e.g. "~1 CPU-hour"), NOT as wide ranges (e.g. "10–300 CPU-hours").

### Part B — Cluster Fit (from cluster documentation)

After computing the estimate in Part A, check whether it fits the cluster:

- Does the job fit within queue processor limits?
- Does memory fit within node limits?
- Is walltime within queue maximums?
- Is the job at risk of pre-emption?
- Are there scheduling or policy constraints?

Report cluster fit as a **separate assessment** from the compute estimate. Example: "The axisymmetric run requires ~1 CPU-hour and ≤1 GB memory. This fits comfortably within the shared queue (max 64 procs, 100 GB/node). Pre-emption risk is low given the short walltime."

Do NOT inflate Part A estimates because of Part B uncertainty. If cluster docs are missing, the Part A estimate is still valid — just note that cluster fit cannot be assessed.

## Step 6 — Risk Identification

Identify risks such as:

- unsupported physics
- missing diagnostics
- resolution constraints
- cluster policy restrictions
- missing input datasets

Conflicts between sources must be **reported, not resolved**.

## Step 7 — Score and Confidence

Start confidence at:

```
0.8
```

Apply penalties:

```
minor assumption: −0.05
missing evidence non-core: −0.10
conflict non-core: −0.15
missing evidence core: −0.25
conflict core: −0.35
uncertain compute estimate: −0.10
```

**Penalty guidance:**

- Missing cluster/HPC docs are **non-core** when compute can be estimated from configs
- Cluster scheduling constraints (pre-emption, queue limits) are **operational risks**, NOT reasons to penalize compute confidence or inflate compute estimates
- **Core** missing evidence = something that blocks understanding whether the model can do the required physics/dynamics/initialization (e.g. no documentation of a required capability)
- Methodological choices left to the researcher (e.g. how to define "stable" vs "unstable" sounding) are **minor assumptions**, not missing evidence
- If the model clearly supports the required capability via documented namelist parameters, do not penalize for missing external benchmarks
- The "uncertain compute estimate" penalty (−0.10) should ONLY be applied when the config parameters themselves are ambiguous (e.g. missing nx/ny/nz). It should NOT be applied when compute is calculable from configs but cluster benchmarks are missing — that is a cluster-fit issue, not a compute issue

Clamp confidence to 0–1.

Assign score:

```
5 = clearly feasible
4 = likely feasible
3 = uncertain
2 = unlikely
1 = blocked
```

**Score guidance:**

- If all required capabilities are documented and supported, and compute is estimable from configs, score should be **4 (likely feasible)** even without cluster docs
- Score 3 should only be used when there is genuine uncertainty about whether the model can support the required capability

## Step 8 — Build Matrices

Create:

### Global Summary Matrix

One row per (RQ, Hypothesis) pair.

### Per-Hypothesis Matrix

Columns:

```
dimension, requirement_or_claim, model_support_assessment, feasibility_constraint, evidence_paths, notes
```

## Step 9 — Identify Unresolved Items

Record:

- missing evidence
- parsing uncertainties
- policy blockers
- unresolved inputs

## Step 10 — Generate Next Actions

Provide:

- evidence gathering steps
- small validation tests
- configuration experiments

These are **suggestions only**.
