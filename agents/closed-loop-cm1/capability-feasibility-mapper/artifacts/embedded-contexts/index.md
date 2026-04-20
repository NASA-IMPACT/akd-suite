# Embedded Contexts

At construction time, the Capability & Feasibility Mapper agent appends two reference documents to its system prompt:

- **`cm1_readme.md`** — CM1 model documentation including namelist reference and model capabilities. Ships with akd-ext at `akd_ext/agents/closed_loop_cm1/context/cm1_readme.md`. Used by the agent to ground capability claims in specific namelist parameters (e.g., `isnd`, `sfcmodel`, `cecd`, `output_cape`). Not reproduced here because the upstream file is large; refer to the akd-ext repo.
- [`cluster-it.md`](./cluster-it.md) — UAH-NSSTC Matrix cluster documentation: structure, SLURM usage, queues, QOS, pre-emption, GPUs, scheduled jobs, applications, interactive commands, and caveats. Used by the agent in Step 5 Part B (cluster fit assessment).

## Why this is called out separately

Most agents' system prompts are self-contained strings. The Capability & Feasibility Mapper's constructor explicitly appends these two files under markdown headings (`## Cluster IT Context` and `## CM1 README Context`) after the base prompt. They are therefore part of the agent's effective context and worth preserving in the artifact, even though they are sourced from separate files in akd-ext.

## Upstream

- CM1 readme: [`akd_ext/agents/closed_loop_cm1/context/cm1_readme.md`](https://github.com/NASA-IMPACT/akd-ext) (not mirrored here; fetch from upstream).
- Cluster IT: see [`cluster-it.md`](./cluster-it.md) in this directory.
