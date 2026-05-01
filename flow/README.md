# AKD Flow

**AKD Flow** is the public multi-agent workflow product. It combines:

- A **FastAPI + LangGraph backend** (from [`NASA-IMPACT/akd-services`](https://github.com/NASA-IMPACT/akd-services)) that hosts the AKD agents and orchestrates them as workflow graphs.
- A **Next.js 15 frontend** (from [`NASA-IMPACT/akd-flow`](https://github.com/NASA-IMPACT/akd-flow)) that provides a natural-language planner and a visual workflow canvas.

**Live deployment:** [flow.akd.odsi.io](https://flow.akd.odsi.io)

## What a user does in Flow

1. **Describe a research goal** in natural language in the Planner (e.g., "Find climate datasets and related code for sea-ice modeling and identify gaps").
2. **Review the generated workflow.** The LLM planner proposes a multi-agent workflow — a graph of published agents connected by edges — and shows it in a card and on the canvas.
3. **Tune parameters.** Each agent node has input fields you can adjust (keywords, spatial bounds, temporal bounds, etc.).
4. **Run the workflow.** Flow streams execution updates: nodes light up as agents start, their outputs unfold in real time, and guardrails annotate each node with pass/fail badges.
5. **Review and share.** Outputs are saved as a chat session. Workflows and outputs can be shared via a public URL; others can import shared workflows into their own workspace.

## Sections

- [`user-guide.md`](./user-guide.md) — end-to-end walkthrough for a working scientist.
- [`architecture.md`](./architecture.md) — how backend and frontend fit together.
- [`backend.md`](./backend.md) — high-level look at `akd-services`.
- [`frontend.md`](./frontend.md) — high-level look at `akd-flow`.
- [`api-overview.md`](./api-overview.md) — conceptual API surface.
- [`streaming.md`](./streaming.md) — how streaming events reach the UI.
- [`deployment.md`](./deployment.md) — Docker / AWS / Helm deployment topology.

## Related

- [`agents/`](../agents/) — the agents Flow publishes.
- [`guardrails/`](../guardrails/) — the safety layer applied around Flow-run agents.
- [`labs/`](../labs/) — the complementary playground where agents are developed before being published to Flow.
