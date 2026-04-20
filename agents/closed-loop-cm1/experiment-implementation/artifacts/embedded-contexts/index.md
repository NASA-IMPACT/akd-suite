# Embedded Contexts

At construction time, the Experiment Implementation agent appends `cm1_readme.md` (from akd-ext `akd_ext/agents/closed_loop_cm1/context/cm1_readme.md`) to its system prompt under a `## CM1 README Context` heading. This provides CM1-specific namelist parameter semantics the agent uses to emit correct `FileEdit` objects.

The file is **not mirrored here** because it is large and tied to CM1 internals; refer to the upstream akd-ext repo for the current version.
