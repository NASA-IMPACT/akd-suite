# AKD Operational Pathways and Governance Model

*Working draft*

---

**Pathways in this document are framed primarily at the team level** — the path a *team working on scientific AI* takes through AKD based on the end goal they are targeting. Different teams pursue different goals (ship a Flow agent, publish an open-source toolkit, build a closed-loop pipeline, deploy AKD on their own cloud, develop a domain-specific guardrail, etc.) and each goal exercises a different subset of AKD's machinery.

The **agent lifecycle** (the nine reference stages an agent travels in §1.3) is the *lower-level mechanic* that team pathways exercise — it is not itself the unit of work. A team may build zero, one, or many agents on the way to their goal; some team pathways skip stages entirely.

Part 1 describes the **team pathways** through AKD and the operational mechanisms (agent lifecycle, runtime topology, delivery surfaces) those pathways exercise. Part 2 describes the **governance model** that runs through them: the layered controls, who owns each, and which controls apply per team pathway.

The two parts are designed to be read together. Every team pathway encounters a specific subset of governance controls; every governance control attaches to one or more team-pathway stages.

A status snapshot at the end separates what is operational today from what is planned.

---

# Part 1 — Operational Pathways for Teams

## 1.1 The AKD ecosystem a team builds on

AKD is a layered ecosystem for designing, validating, and operating scientific AI agents. Each layer abstracts a concern so a team can focus on its domain rather than the plumbing underneath. Knowing which layer the team's pathway primarily interacts with is what makes the rest of this document legible.

The table below describes each layer by **intent** — what the layer lets a team do — rather than by implementation. Implementation detail (frameworks, classes, deployment specifics) lives in each component's repository.

| Layer | What it lets a team do | Components today |
|---|---|---|
| **Design discipline** | Articulate *what* an agent is supposed to do scientifically, *how* it should reason, what evidence it must cite, and what counts as a passing benchmark — before any code is written | CARE methodology (`akd-care`); per-agent artifacts in `akd-suite` |
| **Agent runtime primitives** | Compose an agent without reinventing the reasoning loop, tool invocation, schema validation, streaming, HITL pauses, or guardrail wiring — the concerns every agent shares | `akd-core` |
| **Domain capability catalog** | Reuse published scientific agents and MCP-backed tools (CMR, PDS, ADS, Astroquery, Code Search, Job Management, Gap, CM1 stages) rather than rebuilding them; extend the catalog with new agents and tools | `akd-ext`; hosted MCP servers |
| **Authoring + benchmarking environment** | Compose and iterate an agent in a chat lab, run it against a benchmark suite, inspect per-turn traces and cost estimates — without touching production code or the deployed surface | **AKD Labs** (`labs.akd.odsi.io`) |
| **Production runtime + scientist surface** | Run validated agents and multi-stage workflows against real science, with HITL safeguards at consequential steps, full streaming observability, and a trace log for every call | **AKD Flow** (`flow.akd.odsi.io`); the runtime services and UI behind it (`akd-services`, `akd-flow`) |
| **Knowledge layer** | Read what a published agent actually does — its prompt, reasoning strategy, tool catalog, output format, guardrail composition — without reading source code | `akd-suite` (this repo); per-agent `artifacts/` directories |

Two intent-named surfaces are exposed today, and most teams interact with one or both:

- **AKD Labs** — the authoring surface. Where builders compose, validate, and benchmark agents before they reach Flow. Multi-tenant: org → project → user, with publication scope per agent.
- **AKD Flow** — the scientist-facing surface. Where research happens. Closed-loop workflows render on a planner + canvas as they execute, so the researcher sees what the agent is doing in real time. HITL pauses surface here.

Flow and Labs are separate environments **by design**: validation lives in Labs, production lives in Flow, and promotion between them is a deliberate act (see §1.4 for the current mechanic and §1.5 for the planned data-path version).

The six layers above are the operational subset of the broader **Reference Enterprise Stack** ([`README.md` §Reference Enterprise Stack](./README.md#reference-enterprise-stack)), which adds explicit layers for auth/SSO, multi-registry search & discovery, platform services (sandbox, JupyterHub, SDE APIs), and infrastructure choices (NASA Science Cloud / on-prem / personal cloud) that today's stack only partially fulfills. [`roadmap.md`](./roadmap.md) tracks the gap.

## 1.2 Team pathways through AKD

A team's **end goal** determines the pathway it takes through AKD. Different goals exercise different subsets of the per-agent lifecycle (§1.3), land on different delivery surfaces (§1.6), and encounter different governance gates (Part 2). The table below names the team pathways AKD currently supports or plans to support.

A team's pathway is the path *that team* takes from goal to outcome. The per-agent lifecycle in §1.3 is the lower-level mechanic that some — but not all — of these pathways exercise.

| # | Team pathway | End goal | Lifecycle stages exercised (§1.3) | Primary delivery surface | Status / examples |
|---|---|---|---|---|---|
| **A** | **Domain Agent Team** | Ship a single-purpose scientific agent on Flow (data discovery, code discovery, research synthesis, etc.) | All 9 stages | AKD Flow | **Operational** — CMR, PDS, Astro Search, Code Search, Gap teams |
| **B** | **Closed-Loop Pipeline Team** | Ship a multi-stage scientific workflow with HITL approval gates between stages | All 9 stages, applied per stage; adds workflow composition + inter-stage HITL | AKD Flow (workflow canvas) | **Operational** — CM1 pipeline team (Gap → Capability/Feasibility → Workflow Spec → Experiment Implementation → Research Report) |
| **C** | **Open-Source Toolkit Team** | Publish a reusable agent, tool, or example for the community | CARE → Author → Validate → Formalize → Tag *(skips backend register, surface, deploy)* | GitHub source release | **Operational** — public `akd-core`, `akd-ext`, `akd-suite` releases |
| **D** | **Domain Guardrail Team** | Ship a domain-customized Risk Agent (output guardrail) for a scientific domain | CARE-derive domain risk taxonomy → Formalize as guardrail in `akd-ext` → attach to host agents | Bundled with host agents | **Emerging** — pattern produced by CARE per agent (see §2.2 control 2) |
| **E** | **Tool / MCP Team** | Ship a new scientific MCP server or domain tool that any agent can use | API contract → MCP server build → register in Tools Registry *(planned)* → wire into agents via `allowed_tools` | MCP endpoint + Tools Registry | **Operational** — CMR_MCP, PDS_MCP, ADS, Astroquery, Code Search, Job Management |
| **F** | **Adopter / Self-Host Team** | Run AKD on the team's own infrastructure (personal cloud, on-prem) | Skip authoring; deploy reference stack with own SSO / LLM keys / MCP credentials | Personal cloud or on-prem | **Roadmap** — Compose stack close; SSO + reference deployment pending |
| **G** | **RAG Chatbot Team** | Ship a science-grounded RAG chatbot for a research group | Corpus curation → Conversational RAG Engine config → chatbot UI | Application Components (RAG Chatbot) | **Roadmap** — depends on standalone Conversational RAG Engine |
| **H** | **Researcher / Scientist (consumer)** | Use AKD to do science (the consumer pathway) | Discover agents in Flow → run with own data → consume results | Any delivery surface | **Operational** — current Flow user population |

> **A2A / Agent Marketplace is a core AKD service, not a team pathway.** AKD itself operates the marketplace and agent-skill registry; its governance gates (registry approval, cross-org auth, public-use policy per surface, agent-skill manifest conformance) are part of the AKD governance framework. Teams targeting A2A as a delivery surface do so through Pathways A or B; the marketplace's own governance is enumerated in §1.6 (surface 4) and §2.2 controls 5, 7, 9.

Three notes:

- **Pathways are not exclusive.** A team may run more than one in parallel. The CM1 team runs Pathway B but individual stages also exercise Pathway A patterns. The same `akd-ext` agent can be a Flow-published agent (A), source-visible in a public repo (C), and (when AKD's A2A marketplace ships) reachable as an A2A endpoint at the same time.
- **Researcher / Scientist (Pathway H) is the consumer side.** It does not build agents; it uses them. Most documentation and governance discussion focuses on builder pathways (A–G), but the consumer pathway has its own governance touchpoints (ACL, attestation, audit-log access) and is the reason all the other pathways exist.
- **Pathways vs. delivery surfaces.** §1.6 lists the surfaces an agent can be delivered to. Those surfaces are **outputs** of team pathways, not pathways themselves. Pathway A and B deliver to Flow; Pathway C delivers to GitHub; Pathway E delivers an MCP endpoint; an A2A endpoint is delivered through the AKD-operated A2A marketplace (a core service, not a pathway). A team may target one or more delivery surfaces per pathway.

The bulk of the existing operational documentation describes **Pathway A** because it is the most common and the most fully developed today. §1.4 and §1.5 detail Pathway A's mechanics specifically (Labs → Flow promotion, current and planned). §1.6 enumerates the delivery surfaces. Part 2's governance walk-through (§2.4) is also Pathway A. §2.6 maps the governance gates to each team pathway in turn.

## 1.3 Reference agent lifecycle

The nine stages below are a **reference lifecycle** for one agent. They are exercised *in full* by Pathways A and B (Domain Agent Team, Closed-Loop Pipeline Team), *partially* by Pathways C, D, E, *not at all* by Pathway F (which deploys but does not author) and Pathway H (which consumes). Per-pathway exercise is summarized in §1.2; per-pathway governance is in §2.6.

The reference lifecycle is the canonical longest path; team pathways consume it differently.

Each stage has an owner, defined inputs and outputs, and at least one governance touchpoint.

| # | Stage | Primary Owner | Inputs | Activities | Outputs | Governance touchpoint |
|---|---|---|---|---|---|---|
| 1 | **Author in Labs** | Developer + SME | Domain question, data sources, candidate tools | Compose system prompt, wire MCP servers, configure model and settings, iterate in chat lab | `AgentConfig` row in Labs (system prompt, tools, model, settings, scope) | Project/org RBAC; visibility scope (`project` / `organization`) |
| 2 | **Validate in Labs** | Developer + SME | `AgentConfig`, benchmark suite | Run agent against test suite via Benchmark Runner; inspect `TraceLog` per turn; review token cost estimates | `Run` and `Result` rows; per-turn trace logs; cost estimates | Trace inspection, benchmark grading |
| 3 | **Formalize in `akd-ext`** | Developer | Validated `AgentConfig` | Subclass `OpenAIBaseAgent`; declare `input_schema`, `output_schema`, `config_schema`; copy system prompt as constant; wire `HostedMCPTool`s with `allowed_tools` whitelists; attach guardrails | PR against `NASA-IMPACT/akd-ext` | Code review; schema conformance; guardrail wiring |
| 4 | **Tag and pin** | Developer | Merged `akd-ext` commit | Tag `akd-ext` version; bump pin in `akd-services`'s `pyproject.toml` | `akd-services` PR | Code review |
| 5 | **Register in backend** | Developer | Pinned `akd-ext` | Add `registry.register_agent(agent_class=..., agent_id=...)` in `akd-services`'s `src/api/runtime/app/main.py` startup | Backend startup registers agent at boot | Code review; explicit static registration (no dynamic loading) |
| 6 | **Surface in frontend** | Developer | Registered agent | Add entry to `AVAILABLE_AGENTS` in `akd-flow`'s `src/constants/agents.ts` | Frontend PR | Code review |
| 7 | **Deploy** | Ops Engineer | Both repos merged | Redeploy backend (ECS Fargate / Helm) and frontend; run smoke tests against `flow.akd.odsi.io` | Live agent in Flow planner | Deployment gate; environment review |
| 8 | **Operate** | Ops Engineer + SME | Live agent | LangGraph checkpointer persists run state in PostgreSQL; SSE streams events to UI; HITL pause/resume; guardrails enforce input/output filters; trace logs accumulate | Live workflow runs; trace logs; HITL approvals | HITL gates per agent; guardrails at every call; trace inspection |
| 9 | **Iterate / retire** | Developer + SME | Operational feedback, prompt drift | Loop back to Labs for prompt iteration; re-PR `akd-ext` for material changes; remove from `AVAILABLE_AGENTS` and unregister at backend startup on retirement | New version or removal record | Re-entry through stages 3–7 for any non-trivial change |

Two properties of this lifecycle matter for the governance model that follows. First, **promotion from Labs to Flow is a code path** — not a data path. There is no runtime plugin loader or artifact registry in Flow today; an agent authored in Labs does not automatically appear in Flow. Second, **a developer must keep four artifacts in sync by hand**: the Labs `AgentConfig`, the `akd-ext` Python source, the `akd-services` startup registration, and the `akd-flow` `AVAILABLE_AGENTS` entry. Drift between Labs and the deployed Flow agent is a known friction point.

## 1.4 Labs → Flow promotion (current state) — Pathway A mechanic

This is the mechanic that **Pathway A** (Domain Agent Team) and **Pathway B** (Closed-Loop Pipeline Team) use today. The promotion path is documented explicitly in `labs/publishing-to-flow.md`:

> Today this is a **code path, not a data path**, and future plans move toward an artifact-based workflow.

The seven steps in §1.3 (stages 3–7) are the entire process. There is no out-of-band release surface, no marketplace, no plugin store. Why it works this way today:

- Static registration is simple, debuggable, and fast at request time.
- Agents use complex Python patterns (tool config, output schemas, `check_output` overrides, embedded context files) that do not map cleanly onto the simpler data model Labs stores.
- The published agent set turns over slowly; dynamic loading has not been needed yet.

## 1.5 Labs → Flow promotion (planned)

The artifact-based promotion path is a **planned, not implemented** direction. The target shape is a portable artifact bundle resembling this repo's per-agent `artifacts/` directories:

- A manifest (`index.md` with YAML frontmatter — agent id, class reference, metadata).
- The system prompt, split across context files (role, reasoning strategy, constraints, output format).
- A tool config declaratively naming MCP servers and whitelisted tools.
- Input- and output-schema descriptions.
- Optional embedded context files (e.g., the CM1 README and cluster docs that ship with the CM1 pipeline agents).

What would need to land:

- A serialization format for the bundle.
- A loader in `akd-ext` / `akd-services` that takes a bundle and produces an agent instance.
- A registry service or conventions around a git repo or object store.
- An export UI in Labs.
- A dynamic agent-list endpoint in the Flow backend.

The motivation is single source of truth: an agent's prompt and config live in one place, and Labs → Flow promotion becomes a data operation rather than a code-and-redeploy operation.

## 1.6 Delivery surfaces

Delivery surfaces are **outputs** of team pathways (§1.2), not pathways themselves. Once an agent is formalized in `akd-ext`, it can be routed to one or more delivery surfaces. An agent does not need to ship through all of them — most ship through one or two, and which surfaces a team targets depends on its pathway end goal.

**AKD Labs is not a delivery surface.** Labs is the authoring and benchmarking environment used to validate an agent before it is promoted (see §1.4). The surfaces below are where an agent can land *after* it leaves the authoring environment.

| # | Surface | Primary Audience | Engagement Method | Required Readiness Beyond `akd-ext` Merge | Maintenance Owner | Status |
|---|---|---|---|---|---|---|
| 1 | **AKD Flow** (`flow.akd.odsi.io`) | NASA scientists, end users of NASA Science data | Direct application access; planner + workflow canvas UI; SSE-streamed events; HITL pauses surfaced in UI | `register_agent` in `akd-services/main.py`; entry in `AVAILABLE_AGENTS` in `akd-flow`; backend + frontend redeploy; smoke tests | Ops Engineer + Developer | **Operational** — `flow.akd.odsi.io` is live; published agents include CMR, PDS, Astro Search, Code Search, Gap, and the 5 CM1 stages |
| 2 | **GitHub repositories (open-source framework + agent source)** | External developer teams, NASA dev teams | Public repos, documentation, technical workshops. Anchor repos: [`NASA-IMPACT/akd-core`](https://github.com/NASA-IMPACT/akd-core) for primitives and [`NASA-IMPACT/akd-ext`](https://github.com/NASA-IMPACT/akd-ext) for domain agents. Meta repo [`NASA-IMPACT/akd-suite`](https://github.com/NASA-IMPACT/akd-suite) carries the consumable knowledge layer. | Sanitized prompts and configs; license review; release tagging; the `akd-suite` artifact mirrors upstream `akd-ext` system prompt and tool config | Project Lead + Developer | **Operational** — five working repos (`akd-core`, `akd-ext`, `akd-services`, `akd-flow`, `akd-labs`) plus the `akd-suite` meta repo; `akd-suite` uses the upstream-fidelity convention |
| 3 | **AKD as app in external chat (ChatGPT, Claude)** | Researchers using ChatGPT or Claude | Custom GPT links, project shares, Claude "projects" wired to AKD MCP servers | External-facing prompt review; data-leakage check; org-policy review (e.g., IBM/legal concerns); MCP server access scoping | Developer | **Roadmap** — depends on hosting AKD MCP servers with external-facing auth; not present in the public repo today |
| 4 | **A2A / Agent Marketplace** *(AKD core service)* | Other agent ecosystems and LLM-app developers | Hosted endpoints under an Agent-to-Agent protocol with delegated authentication. AKD itself operates the marketplace and registry; teams reach this surface through Pathways A or B. | Protocol selection (A2A vs. alternatives); agent-skill manifest format; marketplace metadata; cross-org auth — all owned by the AKD platform, not the publishing team | AKD Platform (Ops Engineer + Service Owner) | **Roadmap** — protocol selection pending; marketplace requirements being scoped |
| 5 | **Enterprise Services & Endpoints** | Internal ops engineers, downstream applications | Endpoint sharing, API documentation, SLAs | API contract; quota and rate limits; SLA agreement; security review at endpoint scope | Ops Engineer | **Initial exploration** — requirements being gathered |

Two notes on this set. First, the pathways are **not exclusive** — the same `akd-ext` agent can be a Flow-published agent, source visible in a public repo, and (in the future) an A2A endpoint at the same time, with each surface carrying its own readiness checks. Second, **the readiness column is additive**, not a replacement. Surface-specific checks come on top of the core lifecycle controls — non-prescriptive design, guardrails, HITL, schema conformance, registration — never instead of them. An agent does not skip guardrails because it is "only" being shared as a Claude project.

A note on **AKD Flow** specifically. Flow is the operational surface for what the AKD vision calls **closed-loop scientific workflows** — research cycles within a defined scientific scope, with a curated inventory of MCP-backed tools, an agent that adapts the workflow as evidence comes in, and HITL safeguards at every consequential step. The Closed-Loop CM1 pipeline (Gap → Capability/Feasibility → Workflow Spec → Experiment Implementation → Research Report) is the first such workflow shipped end-to-end. The defining UX choice is that the workflow is rendered on a planner + canvas surface as it executes — so the researcher sees what the agent is doing, in what order, with what tools, against which data, in real time. Flow ships approved agents and registered workflows; ad-hoc authoring lives in AKD Labs.

## 1.7 Runtime topology

Once an agent is registered, every call runs through the same backend topology. The Flow Compose stack (per `flow/deployment.md`):

| Service | Port | Purpose |
|---|---|---|
| postgres | 5432 | Workflow/chat storage; LangGraph checkpoints |
| redis | 6379 | Rate-limit cache |
| migrate | — | Alembic migrations on boot |
| searxng | 8080 | Meta-search backend for web-search tools |
| ollama | 11434 | Local LLM runtime (Granite Guardian; fallbacks) |
| factuality | 8000 | FactReasoner FastAPI service |
| api | 8001 | Main FastAPI backend |
| frontend | 3000 | Next.js, separate process |

In production the backend runs on ECS Fargate behind an ALB (or Helm on Kubernetes); PostgreSQL is RDS; Redis is ElastiCache. NASA-IMPACT infrastructure manages the public hosts.

Every agent call is observable through three channels: **SSE stream events** to the frontend (lifecycle, tool calls, tool results, reasoning summary tokens, partial output, HITL pauses, failures); **LangGraph checkpoints** persisted to PostgreSQL (which is what makes HITL pauses survive process restarts); and **trace logs** in Labs for any benchmark run.

## 1.8 Versioning, deprecation, retirement

There is no semantic version metadata attached to a Flow-registered agent today. Versioning happens at two natural seams: the `akd-ext` git tag (pinned in `akd-services/pyproject.toml`), and the contents of the `AVAILABLE_AGENTS` list. Material change to an agent's prompt or tools requires a new `akd-ext` PR, a new tag, a backend pin bump, and a redeploy — that act is the version boundary.

Retirement is the inverse of stage 5: remove the `register_agent` call, remove the entry from `AVAILABLE_AGENTS`, and redeploy. Trace logs and benchmark runs in Labs persist as the historical record.

## 1.9 Reference Enterprise Stack pathway (stack view)

§1.2–§1.8 describe governance along the **time axis** — the team pathways and the agent lifecycle they exercise. The Reference Enterprise Stack (see [`README.md`](./README.md#reference-enterprise-stack)) describes the **stack axis** — a vertical governance column running alongside a layered platform. Both views describe the same set of decisions; they slice them differently. The stack view is the long-term shape of the platform; today's operational surface fulfills it partially. [`roadmap.md`](./roadmap.md) tracks what is missing per layer.

The stack-axis pathway maps each governance gate to the layer it gates:

| Stack layer | Governance gate | What it decides | Maps to time-axis stage / control |
|---|---|---|---|
| Application Components (AKD SDK) | Security / Public Use Policy Approval | May this surface be exposed to external users? | Stage 7 (Deploy); §2.2 control 9 — extended per surface |
| Auth, Authorization, Protocols | Access Control List | Org / project / user scope; SSO; quota | Stage 1 visibility scope; §2.2 controls 6 + 7 |
| Search & Discovery | Registry Approval Process | What enters each registry; metadata; deprecation | Stage 5 (Register); §2.2 control 5 — only the Agent Registry today |
| Agentic Layer | SME Approval | Agent prompt + tool composition is scientifically valid | Stage 3 (Formalize) PR review; §2.2 control 1 |
| Services (APIs) | Cost Calculator → Budget Approval | Per-call cost; service quotas; cost envelope | No current stage / control — **gap** |
| Science Artifacts & KB | CARE Design Process & Benchmarks | Did the artifact pass CARE discipline + benchmark thresholds? | Stage 1 (Author) + Stage 2 (Validate); §2.2 control 1 — implicit today |
| Infra | Deployment & Environment Review | Where it runs; data residency; secrets | Stage 7 (Deploy); §2.2 control 9 |

End-to-end, the gates apply in the order below — from CARE design at the top of the time axis down to the user invocation at the bottom of the figure. Every decision in the chain narrows what the user is finally allowed to invoke:

1. **CARE Design Process & Benchmarks** *(Science Artifacts & KB)*. `akd-care` (Collaborative Agent Reasoning Engineering) produces design artifacts (role, reasoning strategy, constraints, output format, tool whitelist) and benchmark thresholds. Nothing moves up the stack without passing CARE.
2. **SME Approval** *(Agentic Layer)*. The SME signs off that the composed agent — prompt + tools + RAG configuration — answers the scientific question correctly.
3. **Cost Calculator → Budget Approvals** *(Services)*. Per-call cost projected (LLM inference + sandbox compute + MCP calls + guardrail evaluation); budget owner approves the envelope per project / per agent.
4. **Registry Approval** *(Search & Discovery)*. Artifact published to the appropriate registry — Agents (operational), Tools / Notebooks / Documents-Code (planned). Each registry enforces metadata, lineage, deprecation policy.
5. **Access Control List** *(Auth)*. Scope is bound: who in which org/project sees the agent, what quota each role gets, which MCP credentials are wired.
6. **Security / Public Use Policy Approval** *(Application Components)*. Final gate before exposure on a public-facing surface — AKD UI (Flow), RAG chatbot, traditional search, future A2A endpoints. Privacy review, data leakage, public-use policy.
7. **User**. Runtime governance (guardrails, HITL, schema validation, streaming observability — §2.2 controls 1–4 and 8) takes over from here.

### Gaps the stack view exposes

Three governance gates in the figure don't have a first-class implementation today:

- **Cost Calculator → Budget Approvals.** No current control. Depends on Quota Management at the Auth layer.
- **Multi-registry approval.** §2.2 control 5 covers only the static Agent Registry in `akd-services/main.py`. Tools / Notebook / Documents-Code registries each need their own approval flow.
- **Public Use Policy Approval per surface.** §2.2 control 9 covers `flow.akd.odsi.io`. Each new application surface (RAG chatbot, traditional search, A2A endpoints) needs its own public-use policy gate.

Two further nuances:

- **Scientific Guardrails as platform services.** The figure positions Factuality Detection and Compliance Checking at the Services layer — shared infra any agent can call — alongside the Granite Guardian / Risk Agent composition that today is wired per-agent. Promotion from per-agent wrappers to platform services with explicit SLAs is on the roadmap.
- **Personal Cloud as an Infra option.** The figure shows Personal Cloud alongside NASA Science Cloud and On-Prem. That introduces a data-residency and key-management decision the current operational pathway doesn't model.

See [`roadmap.md`](./roadmap.md) for the layer-by-layer plan and sequencing.

---

# Part 2 — Governance Model

## 2.1 Philosophy

Governance in AKD is **distributed across the runtime**, not concentrated at a single approval gate. An agent's authority to run is the sum of smaller, layered controls: the *non-prescriptive* posture baked into every system prompt, the input/output guardrails wrapped around every call, the HITL pauses authored into the agent's reasoning, the typed schemas enforced at every boundary, the static registration that decides which agents Flow will load at all.

Each control produces an artifact (a system prompt, a guardrail composition, a HITL event in the trace, a Pydantic schema, a `register_agent` line). Together those artifacts form the agent's audit surface.

This shape is deliberate. A single end-of-pipeline gate is brittle and tends to discover the wrong issues at the wrong time. Layered controls surface the right concerns at the right place — design issues during prompting, content issues at the guardrail layer, judgment calls at HITL pauses, contract issues at schema validation, deployment issues at backend registration — and produce a record of how each was resolved.

The controls below are organized around three trustworthy-AKD properties: **non-prescriptive output** (the agent surfaces candidates and evidence; the human researcher decides), **human control** (the human can intervene at any consequential step), and **transparency** (every model call, tool call, and intermediate reasoning step is captured as a stream event and a trace log).

## 2.2 The governance controls

### 1. Non-prescriptive design
- **What it gates:** What the agent is allowed to *say*. Every published AKD agent is explicitly designed to surface candidates, evidence, and caveats — never to recommend, endorse, select, or judge significance.
- **Where it lives:** The agent's system prompt, captured upstream in `akd-ext` and mirrored in `agents/<agent>/artifacts/contexts/constraints.md` in this repo.
- **Owner:** SME and Developer (jointly), authoring the system prompt; reviewers of the `akd-ext` PR.
- **Examples in production:** The CMR CARE Agent "never recommends, selects, or endorses datasets." The PDS Search Agent never executes downloads or password-protected workflows. The Gap Agent never declares novelty, resolves contradictions, or judges feasibility, importance, or significance. The Code Search agent never uses prescriptive language ("best," "recommended," "approved").
- **Lifecycle stage:** Authored in Labs (Stage 1), formalized in `akd-ext` (Stage 3), enforced at every call.
- **Status:** Operational across all six Flow-published agent families.

### 2. Guardrails (input and output)
- **What it gates:** Content that enters and leaves any agent. AKD's guardrails implement the `GuardrailProtocol` and compose with operators: `&` (AND, parallel), `|` (OR, parallel), `>>` (fail-fast, sequential).
- **Owner:** Developer (selects and composes); Project/SME (signs off on the composition).
- **Output artifacts:** Guardrail composition wired via `@guardrail(input_guardrail=..., output_guardrail=...)` or `apply_guardrails(...)`; runtime decisions captured in trace logs.
- **Two providers ship today, each pointed at a different side of the call:**
  - **Granite Guardian — input guardrail (human → agent).** IBM's content-moderation family, run locally via Ollama. Wraps user-supplied input before it reaches the agent: jailbreak, violence, sexual content, profanity, social bias, unethical behavior, harm, plus RAG-specific groundedness/relevance/answer-relevance checks. Single-risk mode (`granite3-guardian:2b/8b`, `granite3.3-guardian:8b` chain-of-thought) parallelizes per-category checks (default concurrency 3); multi-risk mode (`granite-guardian-3.2-5b-multi-harm-GGUF`) does a two-step harmful/safe decision plus harm-category extraction with a configurable score threshold.
  - **Risk Agent — output guardrail (agent → human / downstream).** An LLM-judge that evaluates agent-emitted outputs against the **IBM Risk Atlas** and a **Science Literature Risks** taxonomy (hallucination, attribution gaps, consistency failures, overgeneralization, positivity bias, multidisciplinary gaps). It dynamically generates evaluation criteria, builds a hierarchical importance-weighted DAG, runs `DAGMetric.a_measure()` via DeepEval, and auto-generates a detailed risk report for failed HIGH-importance criteria. Default pass threshold is 0.9. **Domain-customized Risk Agents are themselves CARE artifacts** — for an agent in a specialized domain (heliophysics, planetary science, etc.), the SME and developer derive the domain risk taxonomy during the CARE process, and the resulting Risk Agent configuration ships alongside the agent as part of its artifact bundle.
- **Recommended composition:**
  - **On input** — Granite Guardian on user-supplied content. Cheap, deterministic, no LLM-judge cost.
  - **On output** — Risk Agent (general taxonomy + any domain-customized variant) on agent-emitted content. The general taxonomy catches cross-domain risks; the CARE-derived domain variant catches discipline-specific ones.
  - For agents that need belt-and-braces input filtering (e.g., when user input is also passed back into the output), `granite >> risk_agent` chains them fail-fast.
- **Lifecycle stage:** Stage 3 (formalize) — both general guardrails wired and any domain Risk Agent attached. Enforced at Stage 8 (operate).
- **Status:** Operational. Both providers shipped in `akd-core`/`akd-ext`. Domain-customized Risk Agents are an emerging pattern, produced by CARE per agent.

### 3. Human-in-the-loop (HITL)
- **What it gates:** Every consequential decision. HITL is a first-class capability in `akd-core` (via `HumanTool` and the streaming model), not a feature added on top.
- **Owner:** SME (authors the pause points); Developer (wires `HumanTool`).
- **Mechanism:** The agent emits a `Human input required` stream event mid-run. Run state is persisted by LangGraph's PostgreSQL checkpointer and survives process restarts. When the user responds, the answer is re-injected as a tool result and the run resumes.
- **Where it appears in practice:**
  - *Clarification* — CMR/PDS/Astro/Code Search agents pause for missing scope (variables, spatial bounds, temporal bounds) before searching.
  - *Approval gates* — the Closed-Loop CM1 pipeline pauses between every stage (Gap → Capability/Feasibility → Workflow Spec → Experiment Implementation → Research Report) for researcher approval.
  - *Indirect inference permission* — the CMR agent asks before multi-hop discovery via Semantic Scholar.
  - *Scope expansion* — the Astro Search agent asks before widening search radius or adding an archive.
- **What it does not replace:** Guardrails. HITL is for decisions the agent genuinely wants a human to make; guardrails are for content that should be filtered automatically.
- **Lifecycle stage:** Authored at Stage 1, persisted by the runtime at Stage 8.
- **Status:** Operational across all data-discovery agents and the CM1 pipeline.

### 4. Schema and contract conformance
- **What it gates:** What an agent accepts and what it returns. `InputSchema` and `OutputSchema` are Pydantic v2 bases that enforce docstrings; agents declare `input_schema`, `output_schema`, `config_schema` at class level.
- **Owner:** Developer.
- **Output artifacts:** The schemas themselves, plus runtime validation errors surfaced as `Failed` stream events.
- **Lifecycle stage:** Stage 3 (formalize); enforced at every call.
- **Status:** Operational; required for any `OpenAIBaseAgent` subclass.

### 5. Static agent registry (Flow backend)
- **What it gates:** Whether an agent is loadable in Flow at all. The backend registers agents at startup via explicit `registry.register_agent(...)` calls in `akd-services`'s `main.py`. There is no dynamic discovery and no runtime loader.
- **Owner:** Developer (adds the registration line); reviewer of the `akd-services` PR.
- **Output artifacts:** The registration in `src/api/runtime/app/main.py`; the corresponding `AVAILABLE_AGENTS` entry in `akd-flow/src/constants/agents.ts`.
- **Lifecycle stage:** Stages 5–6.
- **Status:** Operational; this is the only path to Flow today.

### 6. Multi-tenant access control (Labs)
- **What it gates:** Who can see and run an agent in Labs. The Labs data model is org → project → user with RBAC; an `AgentConfig` carries a `visibility_scope` of `project` or `organization`.
- **Owner:** Org admin (sets membership and roles); Project lead (sets `visibility_scope`).
- **Output artifacts:** The `AgentConfig.visibility_scope` field; org/project membership rows.
- **Lifecycle stage:** Stages 1–2.
- **Status:** Operational.

### 7. MCP authorization scope
- **What it gates:** What upstream APIs an agent can reach, and which specific tools on those servers it is permitted to call. Each `HostedMCPTool` is parameterized with a `server_label`, a `server_url` (from an env var), and an `allowed_tools` whitelist. Auth and rate-limit handling live server-side.
- **Owner:** Developer (declares the whitelist); deployment owner (provisions env vars and credentials).
- **Output artifacts:** The tool config in `akd-ext`; the env var configuration in the deployment.
- **Servers wired today:** `CMR_MCP_Server` (`CMR_MCP_URL`), `pds_mcp_server` (`PDS_MCP_URL`), `ADS_Search` (`ADS_SEARCH_MCP_URL`), `Astroquery_MCP_Server` (`ASTRO_MCP_URL`), `Job_Management_Server` (`EXPERIMENT_STATUS_MCP_URL`), the Code Search MCP (`CODE_SEARCH_MCP_URL`).
- **Lifecycle stage:** Stage 3 (declared); Stage 7 (provisioned).
- **Status:** Operational.

### 8. Streaming-event observability
- **What it gates:** Whether anyone can see what an agent did. Every agent exposes `astream()` yielding typed events: `Starting`, `Running`, `Completed`, streaming tokens, thinking/reasoning summaries, partial output, tool calling, tool result, human input required, human response, failed.
- **Owner:** Developer (uses `BaseAgent`); Ops Engineer (operates the stream/persistence pipeline).
- **Output artifacts:** SSE stream to the frontend; trace log rows in Labs (`TraceLog` carries the full request/response, tool calls, reasoning, tokens, cost estimate).
- **Lifecycle stage:** Stage 8.
- **Status:** Operational.

### 9. Deployment and environment review
- **What it gates:** What enters production at `flow.akd.odsi.io` (and `labs.akd.odsi.io`). Container images are deployed via ECS Fargate or Helm; secrets and MCP credentials are environment-managed.
- **Owner:** Ops Engineer.
- **Output artifacts:** CDK / Helm manifests, deployed image tags, smoke-test runs.
- **Lifecycle stage:** Stage 7.
- **Status:** Operational.

## 2.3 Roles and responsibilities

A given person may hold more than one role on a given agent.

- **Subject Matter Expert (SME).** Owns scientific judgment. Authors the system prompt's domain content, defines HITL pause points, validates benchmark outputs in Labs, signs off on the `akd-ext` PR.
- **Developer.** Owns implementation. Composes the agent in Labs, formalizes it as an `OpenAIBaseAgent` subclass, declares schemas, wires guardrails and MCP tools, opens PRs against `akd-ext`, `akd-services`, and `akd-flow`.
- **Ops Engineer.** Owns runtime. Manages deployments, env vars, MCP credentials, infrastructure (ECS/Helm/RDS/ElastiCache), Labs and Flow uptime.
- **Org admin / Project lead (Labs).** Owns access scope in Labs — org and project membership, visibility scope per `AgentConfig`.
- **Code reviewer.** Owns the gate at `akd-ext`, `akd-services`, and `akd-flow` PRs. Ensures schema conformance, guardrail wiring, registration correctness.

## 2.4 Walk-through: a new agent through the model

Consider a hypothetical Heliophysics dataset-discovery agent following the CMR/PDS/Astro pattern.

The developer **authors** the agent in Labs: a system prompt enforcing the non-prescriptive posture, a `HostedMCPTool` pointing at the relevant data-system MCP server with an `allowed_tools` whitelist, and a `HumanTool` pause point for clarifying scope. The agent runs in the chat lab, with traces inspected per turn; benchmark suite runs produce graded `Result` rows.

The developer **formalizes** the agent in `akd-ext`: an `OpenAIBaseAgent` subclass with `input_schema`, `output_schema`, `config_schema`; the system prompt copied as a constant; the tool config declaring the MCP server and whitelist; `apply_guardrails(agent, input_guardrail=granite >> risk_agent)`. The PR goes to `NASA-IMPACT/akd-ext`. After review, the maintainer **tags** a release, **bumps** the pin in `akd-services/pyproject.toml`, **registers** the agent in `main.py`, and **adds** an entry to `AVAILABLE_AGENTS` in `akd-flow`.

The Ops Engineer **deploys** the new backend and frontend. Smoke tests against `flow.akd.odsi.io` confirm the agent appears in the planner, accepts input, calls the MCP server, streams events, and pauses at HITL points.

In **operation**, every call streams typed events to the UI; LangGraph checkpoints persist run state in PostgreSQL; guardrails enforce the `granite >> risk_agent` composition on input and output; the Risk Agent emits a detailed risk report for any failed HIGH-importance criterion; trace logs in Labs accumulate for the benchmark cohort.

Six months later the team requests a new tool. Because tools are declared in `akd-ext`, the agent re-enters Stage 3 — a delta PR through `akd-ext` → tag → pin → redeploy. After the delta lands, the new version is in production.

That is the model in motion. Every decision left a trace; every trace lived under a known owner; nothing was decided by a single end-of-pipeline gate.

## 2.5 Governance artifacts per agent

Versioned and stored across the AKD repos:

- **Authoring artifacts (Labs):** `AgentConfig` row (system prompt, tools, model, settings, scope), `Run` rows, `Result` rows, `TraceLog` rows.
- **Formalization artifacts (`akd-ext`):** `OpenAIBaseAgent` subclass, `input_schema` / `output_schema` / `config_schema`, system-prompt constant, tool config, guardrail composition, git tag.
- **Registration artifacts:** `register_agent` call in `akd-services/main.py`, pin in `akd-services/pyproject.toml`, `AVAILABLE_AGENTS` entry in `akd-flow`.
- **Knowledge layer (this repo):** `agents/<agent>/README.md` and `agents/<agent>/artifacts/` (manifest, contexts/, tools/) — the consumable, code-free description that mirrors the upstream system prompt and tool config.
- **Runtime artifacts:** SSE event streams, LangGraph checkpoints in PostgreSQL, trace logs, guardrail decisions, HITL approval records, Risk Agent risk reports.
- **Deployment artifacts:** CDK / Helm manifests, deployed image tags, env var configuration.

## 2.6 Governance pathways per team end goal

§2.2's nine controls and §2.3's roles apply across all team pathways (§1.2), but they apply **differently** depending on the team's end goal. The Open-Source Toolkit Team (Pathway C) does not encounter the deployment review gate; the Adopter / Self-Host Team (Pathway F) does not encounter SME approval but takes on every enterprise-level control (§2.7); the Researcher / Scientist consumer (Pathway H) only encounters access-control and attestation gates.

The table below maps each team pathway to the governance gates it actually encounters and the additional readiness checks specific to that pathway. **+** means the gate applies; **(–)** means it does not; **+!** means the gate applies in a heightened form for that pathway. (The A2A / Agent Marketplace governance row is recorded under §1.6 surface 4 and §2.2 controls 5/7/9 because the marketplace is an AKD core service, not a team pathway.)

| Gate / readiness | A · Domain Agent | B · Closed-Loop Pipeline | C · Open-Source Toolkit | D · Domain Guardrail | E · Tool / MCP | F · Adopter / Self-Host | G · RAG Chatbot | H · Researcher / Scientist |
|---|---|---|---|---|---|---|---|---|
| 1 · Non-prescriptive design | + | + | + | n/a | n/a | (–) | + | (–) |
| 2 · Input + output guardrails | + | + | + | **builds the guardrail** | n/a | (–) | +! *(corpus risks)* | (–) |
| 3 · Human-in-the-loop | + | +! *(per-stage)* | + | + | n/a | (–) | + | (–) |
| 4 · Schema and contract conformance | + | + | + | + | + | (–) | + | (–) |
| 5 · Static agent registry | + | + | (–) | (–) | (–) *(Tools Registry, planned)* | (–) | + | (–) |
| 6 · Multi-tenant access control (Labs) | + | + | + | + | + | n/a *(self-host)* | + | + |
| 7 · MCP authorization scope | + | + | + | n/a | **defines the MCP scope** | (–) | + | (–) |
| 8 · Streaming-event observability | + | + | n/a *(library)* | + | + | + | + | + *(audit log access)* |
| 9 · Deployment & environment review | + | + | (–) | (–) | + *(MCP deployment)* | +! *(self-deployed)* | + | (–) |
| **CARE Design + benchmarks** | + | + | + | +! *(domain risk taxonomy)* | + *(API contract)* | (–) | +! *(corpus design)* | (–) |
| **Cost Calculator → Budget** *(roadmap)* | + | +! *(long-running)* | (–) | + | + | n/a | + | + *(per-user quota)* |
| **Public Use Policy Approval** | + *(Flow only today)* | + | +! *(public release)* | (–) | (–) | (–) | +! *(chatbot is public)* | (–) |
| **License + sanitization review** | (–) | (–) | +! *(public source)* | (–) | + *(if open-source)* | + *(deployment)* | + *(corpus copyright)* | (–) |
| **Attestation of intended use** | (–) | (–) | (–) | (–) | (–) | (–) | (–) | +! |
| **Vendor / supply-chain governance** *(§2.7 ctrl 16)* | (–) | (–) | + *(deps)* | (–) | + *(MCP deps)* | +! *(LLM, MCP, sandbox)* | + | (–) |
| **Data classification & handling** *(§2.7 ctrl 10)* | + | + | +! *(no embedded data)* | (–) | + *(MCP data)* | +! *(org owns scheme)* | +! *(corpus tier)* | +! *(per-run attestation)* |

### 2.6.1 Per-pathway readiness profile

The table above is summary; each pathway has a primary readiness profile worth naming:

- **Pathway A — Domain Agent Team.** The full §2.2 control set; this is what the doc has been describing. Governance walk-through in §2.4 is Pathway A.
- **Pathway B — Closed-Loop Pipeline Team.** Pathway A plus inter-stage HITL approval gates (between every CM1 stage), pipeline-level benchmarking, cross-stage data flow review, and capacity-planning attention because pipelines run long.
- **Pathway C — Open-Source Toolkit Team.** Skips the backend-registration and deployment-review gates (no `flow.akd.odsi.io` deploy). Adds **license + sanitization review** (no embedded credentials, datasets, or org-private context), **public-use policy approval** for the published artifact, and dependency / supply-chain review. Schema conformance still applies; runtime observability does not (a library has no runtime).
- **Pathway D — Domain Guardrail Team.** The team *builds* the output guardrail rather than wires it. The CARE process produces the **domain risk taxonomy** as a primary artifact (§2.2 control 2). Governance focus: peer review of the risk model, integration test against host agents, and benchmark calibration of the score threshold.
- **Pathway E — Tool / MCP Team.** Different artifact (an MCP server, not an agent). Governance focus: API contract review, security review (auth, rate limit, injection-safety), Tools Registry approval (planned, §1.6 + §2.7 control 12), vendor governance for any third-party dependency, MCP deployment review.
- **Pathway F — Adopter / Self-Host Team.** Does not author. Inherits **all** of §2.7's enterprise controls — data classification, approved-model list, MCP catalog, kill-switch, audit retention, vendor governance, DR/BCP, capacity. The org *is* the platform owner; AKD provides the deployable stack and the hooks (see [`roadmap.md`](./roadmap.md) cross-cutting section).
- **Pathway G — RAG Chatbot Team.** Pathway A plus **corpus governance** (what is in the index, copyright/licensing, freshness policy), heightened groundedness and answer-relevance checks (Granite RAG-specific categories), and public-use policy review because chatbots tend to be exposed broadly.
- **Pathway H — Researcher / Scientist (consumer).** The consumer pathway. Governance is mostly **on the user, not the agent**: do they have access (§2.2 control 6 + SSO), are they attesting to intended use, are they within their data-tier permissions, can they read their own audit logs. Per-call governance (guardrails, HITL, observability) operates as designed but is invisible to most researchers.

**Not a team pathway: A2A / Agent Marketplace.** When AKD ships the marketplace, its governance — agent-skill manifest conformance, cross-org auth, rate-limit + SLA review, per-surface public-use policy — is owned by the AKD platform and applied at registration time, regardless of which builder pathway (A or B) produced the agent. See §1.6 surface 4 for the surface-level readiness and §2.7 controls 14, 16, 18 for the platform-level controls that back it.

### 2.6.2 Implication

Governance does not have a single "right" set of gates. The right set is **the set the team's end goal triggers**. A document or runbook intended to onboard a *team* (rather than describe an *agent*) should pick the pathway and walk through only the gates that apply.

## 2.7 Enterprise operating model

§2.1–§2.5 describe governance for **building and shipping a single agent**. This section names what additionally has to be in place when an **organization adopts AKD** at scale — multiple teams building, owning, and operating multiple agents under shared platform infrastructure, shared compliance posture, and shared budget. Most of these are not in scope for the current operational implementation; they are listed here so the gap is visible to any organization adopting AKD.

The framing shift: AKD's nine controls in §2.2 govern a single agent through its lifecycle. The controls and roles below govern **the platform itself** as it lives inside an organization.

### 2.7.1 Additional controls (beyond §2.2's nine)

Each is a control AKD itself does not currently provide; an adopting organization brings (or builds) it. AKD's job is to **integrate cleanly** with the org function that owns the control — providing the auditable surface area (trace logs, registry approval flows, deployment artifacts, kill-switch primitives, tagged artifacts) those functions need to operate.

| # | Control | What it gates | Status in AKD today |
|---|---|---|---|
| 10 | **Data classification & handling policy** | Which data tiers (Public / Internal / Restricted / Mission-sensitive / ITAR) may flow through which agents, MCPs, and surfaces | Implicit per-MCP credential scoping; no formal tier mapping on agents or surfaces |
| 11 | **Approved model list** | Which LLMs an org permits per data tier (e.g., GPT-4o for Internal; on-prem Ollama only for Restricted) | Implicit per-agent; no org-level registry or attestation |
| 12 | **Approved MCP / tool catalog** | Which third-party MCPs an org permits; review of new ones | Per-agent `allowed_tools` whitelist; no org-level approval flow |
| 13 | **Privacy Impact Assessment (PIA)** | Required gate for any agent processing PII or human-subject research data | Not modeled |
| 14 | **Incident response & kill-switch** | How a misbehaving agent is disabled across all surfaces; oncall escalation; post-mortem record | Today: redeploy. No platform-level kill-switch primitive |
| 15 | **Audit log retention & access** | How long traces / checkpoints / risk reports are kept; who can read them | Trace logs persist in Labs Postgres; retention policy not codified; access not separately scoped |
| 16 | **Vendor / supply-chain governance** | Third-party LLM providers, MCP vendors, sandbox vendors — contracts, SLAs, security attestations, dependency review | Not modeled |
| 17 | **Cost attribution & chargeback** | How platform cost is attributed to teams / projects / cost centers (distinct from per-agent cost ceiling) | Roadmap (Cost Calculator); chargeback model not designed |
| 18 | **Cross-team data isolation** | Project A's agent cannot read Project B's data unless explicitly shared; cross-tenant trace-log isolation | Labs RBAC partial; no cross-project enforcement at runtime; cross-org sharing unmodeled (§2.8) |
| 19 | **Onboarding / offboarding** | When a team joins or leaves: provisioning, deprovisioning, agent ownership transfer, data handover | Not modeled |
| 20 | **Disaster recovery / business continuity** | RTO/RPO for the platform; backup posture; failover; data restoration | NASA-IMPACT infra-level only; no platform-defined RTO/RPO |
| 21 | **Change Advisory Board (CAB) / RFC** | Substantial change approval beyond per-PR review (e.g., new delivery surface, new platform service, deprecation of a registered agent) | Not modeled |
| 22 | **Training & certification** | Who is permitted to author agents, approve PRs, sign off as SME, deploy | Not modeled |
| 23 | **Capacity planning** | Forecasting LLM tokens, sandbox compute, storage; scaling policy; reserved capacity | Ad hoc; no formal model |

### 2.7.2 Additional roles (beyond §2.3's five)

Most of these map to functions an organization already has; AKD does not invent them but needs to integrate with them. The "in §2.3 today?" column shows which are already named in the per-agent model.

| Role | Owns | In §2.3 today? |
|---|---|---|
| **AKD Platform Lead** | The platform end-to-end at the org; SLAs across all services and surfaces | No |
| **Data Steward** *(per data domain)* | Data classification, retention, access policy for a data domain (heliophysics, earth science, planetary, etc.) | No |
| **Security Reviewer (InfoSec)** | SOC 2 / ATO / mission ATO review; vulnerability response; pen-test scoping | No |
| **Privacy Officer** | PIAs, retention, right-to-be-forgotten, data subject requests | No |
| **Model Risk Officer / AI Governance Lead** | Approved-model list, AI risk register, model-card attestations | No |
| **Mission Liaison** | Mission-specific constraints (ITAR, mission data sensitivity, ATO scope, data-rights agreements) | No |
| **Vendor Manager** | Third-party contracts (LLM providers, MCPs, sandbox vendors, observability vendors) | No |
| **Change Advisory Board (CAB)** | Substantial change approval; deprecations | No |
| **Incident Commander** | Oncall, incident response, post-mortems, kill-switch invocation | No |
| **Service Owner per surface** | Flow Owner, RAG Chatbot Owner, A2A Endpoint Owner, etc. — runtime ownership of one delivery surface | No |
| **Finance Approver** | Cost envelope per project / team; chargeback approval | No |
| **Auditor (internal / external)** | Reads logs, registries, decisions; produces compliance attestation | No |
| **Researcher / End User** | Uses agents in scientific work; reports issues; attests intended use | No (implicit) |
| **SME** *(existing)* | Scientific judgment per agent | Yes |
| **Developer** *(existing)* | Implementation per agent | Yes |
| **Ops Engineer** *(existing)* | Runtime per deployment | Yes |
| **Org Admin / Project Lead** *(existing)* | Access scope per Labs project / org | Yes |
| **Code Reviewer** *(existing)* | PR gate per repo | Yes |

### 2.7.3 What this means for AKD itself

AKD's job at the platform level is not to **be** every one of these controls or roles, but to **integrate cleanly** with the org functions that own them. Three principles:

1. **Provide the surface area.** Auditable trace logs, registry approval flows, deployment artifacts, kill-switch primitives, tagged artifacts (data tier, approved-model class, MCP catalog membership) — these are the hooks the org's Data Steward, Privacy Officer, Security Reviewer, etc. need to do their work.
2. **Stay non-prescriptive about org policy.** The same way agents are non-prescriptive about science. AKD does not invent the data classification scheme; it accepts the org's scheme and lets artifacts/agents/surfaces be tagged against it.
3. **Surface gaps explicitly.** Controls 10–23 are not fail-fast in AKD today. An organization adopting AKD into a regulated environment will need to layer them — and the platform should make that layering tractable rather than fight it.

[`roadmap.md`](./roadmap.md) tracks the platform-side work that creates these hooks (tagged artifacts, kill-switch primitive, retention policy, chargeback model, etc.).

## 2.8 Open questions and decisions pending

The artifact-based promotion design decides several questions at once: serialization format, loader location, registry mechanism, Labs export UI, and the dynamic agent-list endpoint. Until that lands, four operational questions remain visible:

**Drift between Labs and Flow.** The Labs `AgentConfig` and the deployed `akd-ext` system prompt can diverge silently. There is no automated reconciliation today.

**Upstream fidelity.** The per-agent artifacts in this repo assert upstream fidelity ("the artifact mirrors the agent's current `akd-ext` system prompt and tool configuration. No invented sections, no paraphrasing"). This is currently a convention, not a CI check.

**Versioning surface.** Flow does not display a version per agent. Today, "the version" is implicit in the `akd-ext` tag pinned in `akd-services/pyproject.toml`. A user-visible per-agent version would help operators correlate trace logs to source.

**Cross-tenant agent sharing in Labs.** `visibility_scope` is `project` or `organization`. Cross-org sharing of an `AgentConfig` is not modeled.

---

## Status snapshot

| Component | Status |
|---|---|
| `akd-core` (BaseAgent, BaseTool, schemas, streaming, HITL, guardrail protocol) | Operational |
| `akd-ext` (OpenAI Agents SDK integration, MCP glue, published agents) | Operational |
| `akd-services` (FastAPI + LangGraph backend) | Operational |
| `akd-flow` (Next.js 15 / React 19 frontend) | Operational |
| `akd-labs` (multi-tenant authoring + benchmarking) | Operational |
| AKD Flow (`flow.akd.odsi.io`) — primary delivery surface | Operational |
| AKD Labs (`labs.akd.odsi.io`) — authoring + benchmarking (not a delivery surface) | Operational |
| GitHub repositories (open-source framework + agent source + `akd-suite` knowledge layer) | Operational |
| AKD as app in external chat (ChatGPT, Claude) — delivery surface | Roadmap |
| Managed Agent Runtime / A2A Marketplace — delivery surface | Roadmap (protocol pending) |
| Enterprise Services & Endpoints — delivery surface | Initial exploration |
| Published agents: CMR, PDS, Astro Search, Code Search, Gap, CM1 (5 stages) | Operational |
| Granite Guardian (single-risk + multi-risk via Ollama) | Operational |
| Risk Agent (DAG-based importance-weighted LLM-judge over Risk Atlas + Science Literature Risks) | Operational |
| HITL via `HumanTool` + LangGraph PostgreSQL checkpointer | Operational |
| SSE streaming of typed `StreamEvent`s | Operational |
| Hosted MCP integration (CMR, PDS, ADS, Astroquery, Code Search, Job Management) | Operational |
| FactReasoner FastAPI service in Flow Compose | Operational |
| Static agent registration in `akd-services/main.py` | Operational |
| Artifact-based Labs → Flow promotion | Planned |
| Wiki artifacts as the operational deliverable (replacing hand-edits in `akd-ext`) | Planned |
| Dynamic agent-list endpoint in Flow backend | Planned |
| Per-agent versioning surface in Flow UI | Open question |
| Automated Labs ↔ `akd-ext` drift detection | Open question |

---

## Where this draft needs your input

This is a working draft. Stakeholder responses received so far are recorded inline; remaining questions follow.

### Resolved

**Q1 — Does the nine-stage lifecycle match operations today, or should stages be merged?** *(Resolved.)* The lifecycle is a **reference**, not a strict per-agent path. Multiple **team pathways** (§1.2) and multiple **delivery surfaces** (§1.6) consume it differently — different teams pursuing different end goals exercise different subsets of the nine stages. §1.2 (team pathways) and §1.3 (the reference lifecycle) are updated to reflect this — keep the nine stages.

**Q2 — Guardrail composition under control 2.** *(Resolved.)* The guardrail composition recommendation in control 2 is now refined: **Granite Guardian on the input** (human → agent), **Risk Agent on the output** (agent → human / downstream). Domain-customized Risk Agents are themselves CARE artifacts — produced by the SME and developer during the CARE process for a given scientific domain and shipped as part of the agent's artifact bundle.

### Remaining

**Q3 — Control consolidation.** Should *Schema and contract conformance* fold into another control, and should *Static agent registry* and *Deployment review* be combined? The nine controls are stable as named, but reviewers have asked whether the set is over-decomposed.

**Q4 — Roles.** Does §2.3's five-role set cover the actual people involved? §2.6 *Enterprise operating model* now names additional roles needed for org-wide adoption (Data Steward, Security Reviewer, Privacy Officer, Mission Liaison, Model Risk Officer, Vendor Manager, Incident Commander, Service Owner per surface, Finance Approver, Auditor, AKD Platform Lead, CAB, Researcher / End User). Stakeholder confirmation needed: are these the right names, are any missing, and which the org expects AKD to integrate with vs. own.

**Q5 — Open-questions priority.** Are the four open questions in §2.8 (drift, upstream fidelity, versioning surface, cross-tenant sharing) the right ones to push on next quarter, or does the artifact-based promotion design (§1.5) subsume them?

**Q6 — Upstream-fidelity CI gate.** Should the convention used by this repo's `agents/<agent>/artifacts/` directories be promoted to a CI gate against the live `akd-ext` source?

**Q7 — Enterprise operating model scope.** §2.7 lists 14 additional org-level controls (10–23). For an organization adopting AKD: which of these does the org expect AKD to *implement*, which does the org expect AKD to *integrate with* (org owns the control), and which are out of scope?

**Q8 — Team-pathway coverage.** §1.2 names eight team pathways (A–H), with the A2A / Agent Marketplace recorded as an AKD core service rather than a team pathway. §2.6 maps governance per pathway. Are all eight actually pathways the project intends to support, or should some be merged or deprioritized? In particular: is **Pathway D (Domain Guardrail Team)** a separate pathway or a sub-step of Pathway A's CARE process? Is **Pathway G (RAG Chatbot Team)** a real planned pathway or speculative?
