# AKD Roadmap

*Working draft*

---

## Purpose

This roadmap captures the gap between AKD's current operational surface (the five working repos plus Flow) and the **Reference Enterprise Stack** — the long-term shape of the platform shown in the AKD Enterprise Stack figure (see [`README.md`](./README.md#reference-enterprise-stack)).

The stack is the north star. Each layer carries a governance gate, and each layer has components that are *operational*, *partial*, or *roadmap* today. This document describes what *partial* and *roadmap* mean per layer, what would have to land to close the gap, and how the layers sequence against each other.

For the operational pathway and governance controls that already exist, see [`pathways-governance.md`](./pathways-governance.md).

## How to read this document

Per-layer sections follow the same shape:

- **Today.** The operational surface.
- **Missing.** Components named in the Reference Enterprise Stack figure that don't exist yet.
- **What it takes.** The work required to land them.
- **Sequencing.** Dependencies on other layers.

A consolidated status snapshot follows at the end, and the open decisions that gate sequencing are listed last.

---

## Layer 1 — Application Components (AKD SDK)

**Today.** Two application surfaces are live: AKD Flow (`flow.akd.odsi.io`) for end users, and AKD Labs (`labs.akd.odsi.io`) for authoring. Both are bespoke applications.

**Missing.**
- **Packaged AKD SDK.** No separately distributed SDK exists. `akd-core` and `akd-ext` are Python libraries; an external developer using AKD primitives must clone or pip-install them directly. A versioned PyPI release with examples, release notes, and a `pip install akd` story is missing.
- **Agent Test / Validation Environment as a standalone product.** Test/validation today is a feature inside Labs. The figure positions it as a top-level application component, suggesting standalone packaging.
- **AI Conversational Search (RAG Chatbot).** Not present in current AKD.
- **Traditional Search Interface.** Not present.
- **A2A / Agent Marketplace (core AKD service).** AKD-operated agent-to-agent marketplace and skill registry, with registration-time governance built into the platform (manifest conformance, registry approval, cross-org auth, rate-limit / SLA, public-use policy per surface). Treated as a core service rather than a team pathway.

**What it takes.**
- Decide AKD SDK packaging boundary (`akd-core` only, `akd-core` + `akd-ext`, or a thin facade re-exporting a stable API).
- Stand up the release pipeline (PyPI; semantic versioning; release notes).
- Build a RAG chatbot reference application that consumes the Conversational RAG Engine (Layer 4).
- Build a search UI that consumes the registries (Layer 3).
- Stand up the A2A / Agent Marketplace service: protocol selection, agent-skill manifest schema (shared with Layer 3 registries and Layer 4 declarative composition), marketplace metadata, cross-org auth (Layer 2 SSO).

**Sequencing.** RAG Chatbot depends on the Conversational RAG Engine (Layer 4). Search UI depends on the registries (Layer 3). Packaged SDK can ship independently. A2A / Agent Marketplace depends on Layer 2 SSO + Layer 3 registries + Layer 4 declarative composition.

---

## Layer 2 — Auth, Authorization, Protocols

**Today.** Labs has org → project → user RBAC (`AgentConfig.visibility_scope` of `project` or `organization`). MCP servers carry their own auth via env-var-managed credentials. Flow has no per-user auth surface today.

**Missing.**
- **SSO integration.** Federation with NASA / institutional identity providers.
- **Quota Management as a service.** Per-org / per-project / per-user quotas, billed against the Cost Calculator.
- **Access Control for non-Labs surfaces.** Flow, RAG chatbot, traditional search, future A2A endpoints all need a unified auth model.
- **Cross-org agent sharing.** `visibility_scope` is `project` or `organization`; cross-org sharing of an `AgentConfig` is not modeled (see `pathways-governance.md` §2.8).

**What it takes.**
- Pick an auth provider (Cognito / Okta / Keycloak / NASA SSO).
- Define an identity model that spans Labs + Flow + future surfaces.
- Build a quota service backed by Redis/Postgres with hooks at the Services layer (Layer 5).

**Sequencing.** SSO is a prerequisite for opening Flow to broader user populations. Quota Management is a prerequisite for the Cost Calculator + Budget Approvals governance gate.

---

## Layer 3 — Search & Discovery

**Today.** A static Agent Registry exists in `akd-services/main.py` (`registry.register_agent(...)`) and is mirrored in `akd-flow` (`AVAILABLE_AGENTS`). There is no Tools, Notebook, or Documents/Code registry.

**Missing.**
- **Tools Registry.** A first-class catalog of MCP tools, domain tools, and function tools with versioning, ownership, and deprecation policy.
- **Notebook Registry.** A catalog of benchmark, example, and reproducibility notebooks.
- **Documents / Code Registry.** A catalog of context documents (e.g., the CM1 README and cluster docs that ship with the CM1 pipeline agents) and code repositories that agents reference.
- **Dynamic discovery.** Today registration is at startup with code redeploys; a dynamic discovery endpoint is on `pathways-governance.md` §1.5 as planned.

**What it takes.**
- Define a registry schema (id, version, owner, scope, lineage, deprecation policy) shared across registry types.
- Stand up storage (Postgres-backed service or git-backed artifact-as-source).
- Build an approval flow per registry type (the Registry Approval Process governance gate).
- Surface the registries in Labs and Flow.

**Sequencing.** Tools Registry is the natural first additional registry — it tightens MCP authorization scope (governance control 7 in `pathways-governance.md`) and informs the Cost Calculator. Notebook + Docs registries can follow.

---

## Layer 4 — Agentic Layer

**Today.** The Agentic (LLM+Tools) Engine is `akd-core` + `akd-ext` (`BaseAgent`, `OpenAIBaseAgent`). The Agent Composition Engine is the LangGraph runtime in `akd-services` plus the planner in `akd-core`. Conversational RAG is not packaged as a standalone engine — RAG behavior is implemented per-agent via tool composition.

**Missing.**
- **Conversational RAG Engine as a separate component.** A reusable engine that any application (especially the RAG Chatbot) can call, with corpus selection, retrieval strategy, citation, and groundedness guardrails wired in.
- **Higher-level Agent Composition surface.** Today composition is code; a declarative composition layer (the planned artifact-based promotion in `pathways-governance.md` §1.5) is missing.

**What it takes.**
- Extract RAG primitives from agents that use them (FactReasoner integration, retrieval-augmented patterns) into a packaged engine in `akd-core` or a new `akd-rag` module.
- Define the composition artifact format — manifest + contexts + tool config + schemas. The same schema should be reusable for the registries in Layer 3.

**Sequencing.** The Conversational RAG Engine is a prerequisite for the RAG Chatbot (Layer 1). Declarative composition is a prerequisite for dynamic registry-driven Flow.

---

## Layer 5 — Services (APIs)

**Today.** LLM Inference is operational (OpenAI via SDK; Ollama local for Granite Guardian; FactReasoner FastAPI service in Flow Compose). Scientific Guardrails are operational (Granite Guardian + Risk Agent + FactReasoner). MCP-backed scientific tools are operational (CMR, PDS, ADS, Astroquery, Code Search, Job Management). There is no JupyterHub, no AI sandbox, and no formalized SDE API surface.

**Missing.**
- **JupyterHub.** A managed notebook service that agents (or scientists exploring agent outputs) can spin up.
- **AI Sandbox (e.g., e2b.dev).** An isolated execution environment for agent-generated code / experiments. Required by the Closed-Loop CM1 pipeline's experiment implementation stage if it scales beyond the current job-management MCP.
- **SDE APIs (Scientific Data Engineering).** A formalized API surface beyond ad-hoc MCP servers; standard contracts for data ingest, transformation, lineage.
- **Compliance Checking as a platform service.** Today compliance/risk is per-agent via the Risk Agent; the figure positions it as a shared service.
- **Bedrock / vLLM inference.** The figure names these alongside Ollama; today only OpenAI + Ollama are wired.

**What it takes.**
- Pick a JupyterHub deployment (Zero-to-JupyterHub on Kubernetes is the obvious path).
- Integrate an AI sandbox provider; define the contract between agent and sandbox.
- Inventory existing MCP servers and propose an SDE API specification.
- Promote Granite Guardian + Risk Agent + FactReasoner from per-agent wrappers to platform-callable services with explicit SLAs.
- Add Bedrock + vLLM as inference backends behind the existing inference abstraction.

**Sequencing.** AI Sandbox is the most pressing for the CM1 pipeline. JupyterHub follows. SDE API formalization can run in parallel.

---

## Layer 6 — Science Artifacts & KB

**Today.** Per-agent artifacts in `akd-suite` (under `agents/<agent>/artifacts/`) implement the upstream-fidelity convention — context files and tool configs mirror upstream `akd-ext`. The CARE (Collaborative Agent Reasoning Engineering) methodology lives in `akd-care`.

**Missing.**
- **Artifact registry with metadata schema.** A queryable catalog of artifacts beyond the current convention-based directory structure.
- **Lineage tracking.** Which artifact informed which agent / tool; which benchmark validated which version.
- **KB search across artifacts.** Semantic search over the artifact corpus.
- **CARE design conformance as a CI gate.** Today upstream fidelity is a convention, not a CI check (see `pathways-governance.md` §2.8).

**What it takes.**
- Define an artifact metadata schema (frontmatter pattern; lineage links; benchmark references).
- Build a search index over artifacts.
- Add CI to verify artifact ↔ upstream `akd-ext` parity.

**Sequencing.** The metadata schema is a prerequisite for the Documents / Code Registry in Layer 3. KB search consumes both.

---

## Layer 7 — Infra

**Today.** NASA-IMPACT infrastructure (ECS Fargate / Helm on Kubernetes); RDS Postgres; ElastiCache Redis. Local Ollama for Granite Guardian. On-prem inference partly available.

**Missing.**
- **Personal cloud option.** A "bring your own keys / bring your own infrastructure" deployment mode.
- **Reference deployment for on-prem.** A documented, repeatable on-prem stand-up beyond the current Compose file.
- **Multi-region / data-residency story.** Required for non-US data and for missions with residency constraints.

**What it takes.**
- Package a personal-cloud deployment (Docker Compose + env template) — the Flow Compose stack is close to this already.
- Document an on-prem reference (Helm charts, secrets management, network topology).
- Add a residency policy field to the deployment artifact.

**Sequencing.** Personal cloud unlocks external developer adoption. On-prem unlocks mission-specific deployments. Both depend on the SSO model from Layer 2.

---

## Cross-cutting: Per-agent governance gaps

Three governance gates in the Reference Enterprise Stack figure don't yet have a first-class implementation. These extend the nine per-agent controls in `pathways-governance.md` §2.2:

1. **Cost Calculator → Budget Approvals.** Per-call cost projection (LLM inference + sandbox compute + MCP calls + guardrail evaluation) and per-project budget envelope. Depends on Quota Management (Layer 2). Owners: Project Lead + Finance.
2. **Multi-registry approval.** Today only the Agent Registry has an approval flow. Tools / Notebook / Documents-Code registries each need one. Depends on Layer 3 registry rollout.
3. **Public Use Policy Approval per surface.** Today deployment review covers `flow.akd.odsi.io`. Each new application surface (RAG chatbot, traditional search, A2A endpoints) needs its own public-use policy gate.

These should land as new entries in `pathways-governance.md` §2.2 when implemented.

## Cross-cutting: Enterprise operating model (platform hooks)

`pathways-governance.md` §2.7 names 14 additional controls (10–23) that an organization adopting AKD will need on top of the per-agent set. AKD does not implement them; AKD provides the **hooks** the org's existing functions (Data Steward, Security Reviewer, Privacy Officer, Mission Liaison, etc.) need to operate them. The platform-side work to expose those hooks is roadmap material:

| Hook | What AKD needs to expose | Enables which org control |
|---|---|---|
| **Data-tier tags on artifacts and surfaces** | A `data_tier` field on agents, MCPs, and delivery surfaces that the org's Data Steward can use to assert tier compatibility | Control 10 (data classification), Control 18 (cross-team isolation) |
| **Approved-model attestation** | A `model_class` tag on `AgentConfig` that maps to the org's approved-model list; runtime check at agent boot | Control 11 (approved models) |
| **MCP catalog membership** | A registry-side flag indicating which MCPs are org-approved; enforcement at `allowed_tools` resolution | Control 12 (approved MCP / tool catalog) |
| **PIA artifact slot** | A required PIA reference per agent that processes flagged data tiers | Control 13 (PIA) |
| **Kill-switch primitive** | A platform API to disable an agent across all surfaces in one call (vs. redeploy) | Control 14 (incident response) |
| **Trace-log retention policy field** | A configurable retention horizon per Labs project / Flow deployment, with access scopes separate from operational scopes | Control 15 (audit retention) |
| **Vendor manifest** | A declared list of third-party LLM, MCP, and sandbox vendors with their attestation references | Control 16 (vendor / supply-chain) |
| **Cost-attribution tags** | Cost-center / project / team tags on every billed call (LLM, sandbox, MCP) feeding the Cost Calculator | Control 17 (chargeback) |
| **Project lifecycle hooks** | Provisioning / deprovisioning APIs for Labs projects and Flow agent ownership | Control 19 (onboarding / offboarding) |
| **Backup posture documentation** | Documented RTO/RPO for Postgres (workflow / chat), Redis, trace logs; restore runbook | Control 20 (DR / BCP) |
| **Change-record artifact** | A CAB-readable change record per `akd-ext` tag, registry mutation, and deployment | Control 21 (CAB / RFC) |
| **Authoring permissions** | A permissions surface beyond Labs RBAC: who can author, who can SME-approve, who can deploy | Control 22 (training / certification) |
| **Capacity telemetry** | Token / sandbox / storage telemetry exposed for forecasting | Control 23 (capacity planning) |

Each row above is a platform feature, not a policy decision. The policy decision belongs to the adopting organization; the platform feature unblocks it.

**Sequencing note.** Many of these hooks share substrate with the per-agent gaps above. Cost-attribution tags reuse the Cost Calculator infrastructure. Data-tier tags share the artifact metadata schema being designed for Layer 6. The vendor manifest and approved-model attestation share the same registry pattern as Tools and Agents in Layer 3. Treat the enterprise hooks as *additional fields* on artifacts and APIs being built for the per-agent stack — not as a parallel system.

## Cross-cutting: Delivery surfaces

`pathways-governance.md` §1.6 names five delivery surfaces. Two are operational; three are roadmap items that depend on stack-layer work:

- **AKD as app in external chat (ChatGPT, Claude).** Depends on hosting AKD MCP servers with external-facing auth (Layer 2 SSO + Layer 5 SDE APIs).
- **A2A / Agent Marketplace** *(core AKD service, not a team pathway).* AKD itself operates the marketplace; registration-time governance (manifest conformance, registry approval, cross-org auth, rate-limit / SLA, public-use policy per surface) is part of the platform. Depends on protocol selection (A2A or alternative), agent-skill manifest format (shares schema with Layer 3 registries and Layer 4 declarative composition), marketplace metadata, cross-org auth (Layer 2).
- **Enterprise Services & Endpoints.** Depends on SDE API formalization (Layer 5), SLA model, and quota infrastructure (Layer 2).

---

## Status snapshot

| Layer / item | Status | Sequencing dependency | Notes |
|---|---|---|---|
| **L1: Flow UI / Labs UI** | Operational | — | `akd-flow`, `akd-labs` |
| **L1: Packaged AKD SDK** | Roadmap | — | PyPI release of `akd-core` + `akd-ext` |
| **L1: RAG Chatbot** | Roadmap | L4 RAG Engine | Reference application |
| **L1: Traditional Search UI** | Roadmap | L3 registries | |
| **L2: Labs RBAC** | Operational | — | Org → project → user |
| **L2: SSO** | Roadmap | — | Provider TBD |
| **L2: Quota Management** | Roadmap | L2 SSO | Prerequisite for Cost Calculator |
| **L2: Cross-org sharing** | Open | L2 SSO | See `pathways-governance.md` §2.8 |
| **L3: Agent Registry** | Operational | — | Static, in `akd-services/main.py` |
| **L3: Tools Registry** | Roadmap | — | First additional registry to land |
| **L3: Notebook Registry** | Roadmap | L3 Tools Registry | |
| **L3: Documents / Code Registry** | Roadmap | L6 metadata schema | |
| **L3: Dynamic discovery** | Planned | L4 declarative composition | See `pathways-governance.md` §1.5 |
| **L4: Agentic Engine** | Operational | — | `akd-core` + `akd-ext` |
| **L4: Agent Composition (LangGraph)** | Operational | — | In `akd-services` |
| **L4: Declarative composition (artifact-based)** | Planned | — | See `pathways-governance.md` §1.5 |
| **L4: Conversational RAG Engine** | Roadmap | — | Standalone packaging |
| **L5: LLM Inference (OpenAI, Ollama)** | Operational | — | |
| **L5: LLM Inference (Bedrock, vLLM)** | Roadmap | — | |
| **L5: Scientific Guardrails (per-agent)** | Operational | — | Granite Guardian, Risk Agent, FactReasoner |
| **L5: Guardrails as platform services** | Roadmap | — | Promote per-agent → shared SLA |
| **L5: MCP-backed scientific tools** | Operational | — | CMR, PDS, ADS, Astroquery, Code Search, Job Mgmt |
| **L5: JupyterHub** | Roadmap | — | Zero-to-JupyterHub on K8s |
| **L5: AI Sandbox** | Roadmap | — | e.g., e2b.dev |
| **L5: Formalized SDE APIs** | Roadmap | — | |
| **L6: Per-agent artifacts (convention)** | Operational | — | Upstream-fidelity convention |
| **L6: Artifact metadata schema** | Roadmap | — | Prerequisite for L3 Docs Registry |
| **L6: Artifact lineage tracking** | Roadmap | L6 metadata schema | |
| **L6: KB search** | Roadmap | L6 metadata schema | |
| **L6: CARE-CI parity gate** | Open | — | See `pathways-governance.md` §2.8 |
| **L7: NASA Science Cloud + on-prem (NASA-IMPACT)** | Operational (partial) | — | ECS Fargate / Helm |
| **L7: Personal cloud option** | Roadmap | L2 SSO | Compose stack already close |
| **L7: On-prem reference deployment** | Roadmap | — | Helm charts, secrets |
| **L7: Data residency policy** | Roadmap | — | |
| **Governance: Cost Calculator + Budget** | Roadmap | L2 Quota | New control |
| **Governance: Multi-registry approval** | Roadmap | L3 registries | New control |
| **Governance: Public Use Policy per surface** | Roadmap | L1 surfaces | New control |
| **Surface: ChatGPT / Claude apps** | Roadmap | L2 SSO + L5 SDE APIs | `pathways-governance.md` §1.6 |
| **Core service: A2A / Agent Marketplace** | Roadmap | L3 registries + L4 composition + L2 SSO | AKD-operated; protocol pending; not a team pathway |
| **Surface: Enterprise Endpoints** | Initial exploration | L5 SDE APIs + L2 quota | |

---

## Open decisions

These shape sequencing and aren't yet committed:

- **AKD SDK packaging boundary.** `akd-core` only, `akd-core` + `akd-ext`, or a thin facade that re-exports a stable API.
- **Auth provider.** NASA SSO, Cognito, Okta, Keycloak — affects every Layer 2+ component.
- **Registry storage.** Postgres-backed service, git-backed (artifact-as-source), or hybrid.
- **Composition artifact format.** Decided once for declarative composition (Layer 4) and registries (Layer 3); they should share a schema.
- **Sandbox provider.** External (e2b.dev, Replicate) vs. self-hosted (Firecracker, gVisor).
- **A2A protocol.** A2A vs. alternatives — affects Layer 4 manifest format and the marketplace pathway.
- **Cost model.** Per-call only, or also per-token / per-tool; how chargebacks attribute across orgs.

These should be revisited in `pathways-governance.md` §2.8 as they get resolved.
