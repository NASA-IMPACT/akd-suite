# AKD Reference Enterprise Stack

*Working draft*

The Reference Enterprise Stack is the long-term shape of AKD as a platform — a layered architecture with governance controls aligned to each layer. The lifecycle and team pathways describe how an agent or team moves *through* AKD; the stack describes **what AKD is**.

Today's operational surface fulfills this stack partially. [`roadmap.md`](./roadmap.md) tracks what is missing per layer.

> The canonical Enterprise Stack figure (credit: Ajinkya Kulkarni) is pending. Until it lands, the layer mapping table below is the authoritative reference.

## Layer mapping

| # | Stack layer | Current AKD component(s) | Status |
|---|---|---|---|
| 1 | **Application Components (AKD SDK)** | `akd-flow` (Flow UI); `akd-labs` (test/benchmark UI); A2A / Agent Marketplace *(core AKD service, on roadmap)* | **Partial** — packaged AKD SDK, RAG chatbot, search UI, A2A / Agent Marketplace on roadmap |
| 2 | **Auth, Authorization, Protocols** | Labs RBAC (org → project → user); per-MCP credentials | **Partial** — SSO, quota service, public-surface auth on roadmap |
| 3 | **Search & Discovery** | Static Agent Registry; `AVAILABLE_AGENTS` in `akd-flow` | **Partial** — Agent Registry only; Tools / Notebook / Documents-Code on roadmap |
| 4 | **Agentic Layer** | `akd-core`, `akd-ext`; LangGraph composition in `akd-services` | **Partial** — engine + composition operational; standalone Conversational RAG engine on roadmap |
| 5 | **Services (APIs)** | LLM inference; guardrails (Granite Guardian, Risk Agent, FactReasoner); hosted MCPs (CMR, PDS, ADS, Astroquery, Code Search, Job Management) | **Partial** — JupyterHub, AI sandbox, SDE APIs, guardrails-as-platform-services on roadmap |
| 6 | **Science Artifacts & KB** | `akd-suite` per-agent artifacts; `akd-care` (CARE methodology) | **Partial** — metadata schema, lineage, KB search, CARE-CI gate on roadmap |
| 7 | **Infra** | NASA-IMPACT infrastructure; on-prem Ollama | **Partial** — personal cloud, on-prem reference deployment, residency policy on roadmap |

## Governance aligned to the stack

Each layer carries a governance gate. Full pathway, owners, and the nine operational controls in [`pathways-governance.md`](./pathways-governance.md).

| Layer | Governance gate | Status |
|---|---|---|
| Application Components | Security / Public Use Policy Approval | Operational for Flow; per-surface review on roadmap |
| Auth | Access Control List | Operational (Labs RBAC); roadmap (SSO, public-surface auth) |
| Search & Discovery | Registry Approval Process | Operational for Agent Registry; roadmap for others |
| Agentic Layer | SME Approval | Operational |
| Services | Cost Calculator → Budget Approval | Roadmap |
| Science Artifacts & KB | CARE Design Process & Benchmarks | Operational |
| Infra | Deployment & Environment Review | Operational |

For per-team-pathway governance see [`pathways-governance.md` §2.6](./pathways-governance.md); for enterprise-wide controls (data classification, approved-model list, kill-switch, audit retention, chargeback) see [§2.7](./pathways-governance.md).

The **A2A / Agent Marketplace** sits at Layer 1 as a core AKD-operated service — not a team pathway. Its registration-time gates (agent-skill manifest conformance, registry approval, cross-org auth, rate-limit / SLA, public-use policy per surface) are part of the AKD governance framework regardless of which builder pathway produced the agent. See [`governance.md` §A2A / Agent Marketplace](./governance.md) and [`pathways-governance.md` §1.6](./pathways-governance.md) surface 4.
