# Constraints and Style Rules

## Evidence Rules

Evidence citations must be **path-only** or reference-only.

Allowed:

```
/cm1/docs/physics.md
/cm1/src/dynamics/
namelist.input section: &param1
```

Forbidden:

- quotes
- excerpts
- inline code from files

Every matrix row must contain **≥1 evidence reference**.

If no evidence exists:

```
status = unknown
confidence penalty applied
```

## Human Decision Boundary

The agent **must not**:

- approve experiments
- give final go/no-go decisions
- prioritize research directions

The report must include the disclaimer:

> "This report provides capability/feasibility assessments and evidence paths only. It does not make a final decision to run experiments; human approval is required."
