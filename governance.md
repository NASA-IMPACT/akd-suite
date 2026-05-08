# AKD Governance

*Working draft*

Governance in AKD is **distributed across the runtime** — layered controls aligned to each stack layer rather than a single end-of-pipeline gate. This document is the team-facing summary; the full model, per-control owners, and pathway walk-throughs are in [`pathways-governance.md`](./pathways-governance.md).

## Runtime — applied to every agent call

- **Non-prescriptive design** — agents surface candidates and evidence; never recommend, select, or judge
- **Input guardrails** — **Granite Guardian** on user input (jailbreak, harm, social bias, RAG groundedness)
- **Output guardrails** — **Risk Agent** on agent output. Domain-customized variants are CARE artifacts shipped per agent
- **Human-in-the-loop** — `HumanTool` pause points for clarification, approval, scope expansion
- **Schema conformance** — typed input / output / config schemas enforced at every boundary
- **Streaming observability** — typed `StreamEvent`s to UI and trace logs

## Pre-deployment gates — by stack layer

Each layer of the [Reference Enterprise Stack](./reference-enterprise-stack.md) carries a gate before an agent reaches it in production:

| Layer | Gate |
|---|---|
| Science Artifacts & KB | CARE Design Process + Benchmarks |
| Agentic Layer | SME Approval |
| Services | Cost Calculator → Budget Approval *(roadmap)* |
| Search & Discovery | Registry Approval |
| Auth | Access Control List |
| Application Components | Public Use Policy Approval |
| Infra | Deployment & Environment Review |

Status per gate is in [`reference-enterprise-stack.md` §Governance aligned to the stack](./reference-enterprise-stack.md#governance-aligned-to-the-stack).

## Pathway-aware

Different team pathways encounter different gates. The Open-Source Toolkit Team skips deployment review; the Adopter / Self-Host Team takes on every enterprise control; the Researcher / Scientist (consumer) pathway only encounters access-control and attestation. Per-pathway map in [`pathways-governance.md` §2.6](./pathways-governance.md).

## A2A / Agent Marketplace — core service, not a pathway

The A2A / Agent Marketplace is a **core AKD service** with its governance built into the platform, not a team pathway. AKD itself operates the marketplace and agent-skill registry; the gates that apply at registration time — agent-skill manifest conformance, registry approval, cross-org auth, rate-limit / SLA, public-use policy per surface — are owned by the AKD platform regardless of which builder pathway (A or B) produced the agent. See [`pathways-governance.md` §1.6](./pathways-governance.md) surface 4 and §2.2 controls 5, 7, 9.

## Enterprise operating model

When an organization adopts AKD across teams, additional org-level controls apply: data classification, approved-model list, MCP catalog approval, kill-switch, audit retention, vendor governance, chargeback, cross-team isolation, DR/BCP, change advisory, capacity planning. AKD provides the hooks; the organization owns the policy. Full list and roles in [`pathways-governance.md` §2.7](./pathways-governance.md).
