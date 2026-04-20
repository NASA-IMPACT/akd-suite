# AKD Labs

**AKD Labs** is a multi-tenant agent development and benchmarking platform. It's where experimental agents are built, debugged, tested interactively, and validated — before they graduate to Flow.

**Live deployment:** [labs.akd.odsi.io](https://labs.akd.odsi.io)

Source: [`NASA-IMPACT/akd-labs`](https://github.com/NASA-IMPACT/akd-labs).

## What Labs is

A FastAPI + Next.js web application with a PostgreSQL-backed multi-tenant model (organizations → projects → users, with role-based access control). Labs provides:

- **Agent authoring UI** — create an agent by filling in its name, description, system prompt, model, tool configuration, and model settings.
- **Chat lab** — interact with an agent in a chat interface and see its responses in real time.
- **Trace log** — every chat turn is recorded with full fidelity: system prompt, tool inputs and outputs, model reasoning summary, token usage, and estimated cost.
- **Benchmark runner** — run an agent against a test suite and grade results; compare across agents or agent versions.
- **Publication scope** — once an agent is ready, toggle its `visibility_scope` to `project` or `organization`, track who published it, and make it discoverable within the tenant.

## What Labs is NOT

- Not a production runtime. Agents that need to be exposed to users on Flow graduate via the publishing process described in [`publishing-to-flow.md`](./publishing-to-flow.md).
- Not tied to the Flow backend. Labs is its own app on its own domain.

## Sections

- [`architecture.md`](./architecture.md) — multi-tenant model, agent config object, trace log, benchmark runner, RBAC.
- [`user-guide.md`](./user-guide.md) — using the Labs UI.
- [`agent-development-lifecycle.md`](./agent-development-lifecycle.md) — the create → chat-test → trace → iterate → publish loop.
- [`benchmarking.md`](./benchmarking.md) — how benchmark runs work, result grading, cost tracking.
- [`publishing-to-flow.md`](./publishing-to-flow.md) — how a Labs agent graduates to Flow (current reality + planned artifact workflow).

## Related

- [`flow/`](../flow/) — the production product where published agents are exposed to users.
- [`agents/`](../agents/) — the public catalog of Flow-published agents.
- [`frameworks/akd-ext/`](../frameworks/akd-ext/) — the code-level patterns that Labs-authored agents follow.
