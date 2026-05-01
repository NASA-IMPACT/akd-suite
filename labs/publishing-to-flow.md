# Labs → Flow — Publishing

Moving an agent from "validated in Labs" to "available on Flow" is the final step of the development lifecycle. Today this is a **code path, not a data path**, and future plans move toward an artifact-based workflow. This page documents both.

---

## Current state (as of April 2026)

There is **no runtime plugin loader or artifact registry** in Flow. An agent authored in Labs does not automatically appear in Flow. The current promotion process is:

1. **Formalize the agent as Python** — an `OpenAIBaseAgent` subclass in `akd-ext`, with explicit schemas, a system prompt constant, a config class, and any required tool plumbing. The Labs-authored system prompt is copy-pasted in.
2. **Open a PR** against [`NASA-IMPACT/akd-ext`](https://github.com/NASA-IMPACT/akd-ext) on the active branch.
3. **Merge and tag** a version (or at least reach a stable commit).
4. **Update `akd-services`'s `pyproject.toml`** to pin the new `akd-ext` git rev.
5. **Register the agent** in `akd-services`'s `src/api/runtime/app/main.py` startup code: `registry.register_agent(agent_class=MyAgent, agent_id="my_agent")`.
6. **Update the frontend's agent list** in `akd-flow`'s `src/constants/agents.ts` (`AVAILABLE_AGENTS`) to surface the new agent in the UI — this is a separate PR.
7. **Redeploy** both backend and frontend.

A developer has to keep the Labs agent config, the akd-ext Python source, the backend registration, and the frontend AVAILABLE_AGENTS entry all in sync by hand.

### Why it works this way today

- Static registration is simple, debuggable, and fast at request time.
- Agents use complex Python patterns (tool config, output schemas, `check_output` overrides, embedded context files) that don't map cleanly onto the simpler data model Labs stores.
- There has not been a strong need for dynamic loading yet; the published agent set turns over slowly.

### Friction

- Any Labs → Flow trip takes a full code change + review + redeploy cycle.
- The Labs UI and akd-ext can drift — an agent's prompt in Labs may diverge from the one deployed in Flow without anyone noticing.
- The frontend agent list is hand-maintained.

---

## Planned: artifact-based promotion

**Status: planned / not yet implemented.** The target state below is a design direction, not an available feature.

The idea is to replace the "edit akd-ext and redeploy" path with a **portable artifact bundle** that Flow can load at runtime or during build.

### What an artifact would look like

A portable bundle conceptually resembling this docs hub's agent artifacts (see [`../agents/`](../agents/)): a directory containing:

- A manifest (`index.md` with YAML frontmatter — agent id, class reference, metadata).
- The system prompt, split across context files (role, reasoning strategy, constraints, output format).
- A tool config declaratively naming MCP servers and whitelisted tools.
- An input-schema description (JSON Schema or Pydantic-equivalent).
- An output-schema description.
- Optional embedded context files (for agents like the CM1 pipeline that ship supplementary docs).

This bundle is the same shape as the knowledge layer this docs hub carries — which is the point: the docs hub's artifacts could **become** the deliverable.

### How promotion would work

1. Labs author finishes validating the agent.
2. Labs exports it as an artifact bundle (ZIP / tarball / git subtree).
3. The bundle is published to a registry (shared git repo, S3 bucket, or dedicated service).
4. Flow's backend loads the bundle at build time (or dynamically, depending on deployment preferences) and registers it via the generic registry APIs.
5. The frontend fetches the available-agents list from a Flow backend endpoint rather than a hardcoded file.

### What would need to land

- A serialization format for the bundle (this docs hub's directory structure is a plausible candidate).
- A **loader** in akd-ext / akd-services that takes a bundle and produces an agent instance.
- A **registry service** or conventions around a git repo / bucket.
- An export UI in Labs that generates compliant bundles.
- A dynamic agent-list endpoint in the Flow backend.

### Why it matters

- Single source of truth for an agent's prompt and config.
- Labs → Flow promotion becomes a data operation, not a code + redeploy operation.
- This docs hub's agent artifacts become operational, not just documentary.

---

## What to do today

If you're promoting an agent from Labs to Flow right now, follow the "Current state" steps above and open PRs against both `akd-ext` and `akd-flow`. If you find rough edges, capture them — they will inform the artifact-based design.

## Related

- [`architecture.md`](./architecture.md) — Labs's own data model.
- [`agent-development-lifecycle.md`](./agent-development-lifecycle.md) — where publishing fits in the development loop.
- [`../frameworks/akd-ext/build-a-custom-agent.md`](../frameworks/akd-ext/build-a-custom-agent.md) — how to shape a Labs prompt as a published agent.
- [`../agents/`](../agents/) — what the docs hub's artifact layer looks like today (the structural prototype for the planned promotion format).
