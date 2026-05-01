# akd-ext — Agents Index

All published AKD agents registered at Flow backend startup, with their agent id and deep links into the per-agent knowledge artifact.

| Agent class | Agent id (Flow) | Artifact |
| --- | --- | --- |
| `CMRCareAgent` | `cmr_care` | [`agents/cmr/`](../../agents/cmr/) |
| `PDSSearchAgent` | `pds_search` | [`agents/pds/`](../../agents/pds/) |
| `CodeSearchCareAgent` | `code_search_care` | [`agents/code-search/`](../../agents/code-search/) |
| `AstroSearchAgent` | `astro_search` | [`agents/astro-search/`](../../agents/astro-search/) |
| `GapAgent` | `gap_agent` | [`agents/gap/`](../../agents/gap/) |
| `CapabilityFeasibilityMapperAgent` | `capability_feasibility_mapper` | [`agents/closed-loop-cm1/capability-feasibility-mapper/`](../../agents/closed-loop-cm1/capability-feasibility-mapper/) |
| `WorkflowSpecBuilderAgent` | `workflow_spec_builder` | [`agents/closed-loop-cm1/workflow-spec-builder/`](../../agents/closed-loop-cm1/workflow-spec-builder/) |
| `ExperimentImplementationAgent` | `experiment_implementation` | [`agents/closed-loop-cm1/experiment-implementation/`](../../agents/closed-loop-cm1/experiment-implementation/) |
| `ResearchReportGeneratorAgent` | `research_report_generator` | [`agents/closed-loop-cm1/research-report-generator/`](../../agents/closed-loop-cm1/research-report-generator/) |

## Not currently published to Flow

- `InterpretationPaperAssemblyAgent` — implemented in akd-ext but not registered at Flow backend startup. Lives in the codebase at `akd_ext/agents/closed_loop_cm1/interpretation_paper_assembly.py`. If / when this agent is wired into Flow, it will gain a per-agent directory under [`agents/closed-loop-cm1/`](../../agents/closed-loop-cm1/).

## Note on Flow planner usability

Several of the published agents (Gap and the four CM1 stages) carry `INTERNAL ONLY — Do NOT select this agent in planner workflows` in their descriptions. They are **registered** with Flow and invokable via Flow's single-agent chat endpoint, but the Flow planner is instructed not to pick them à la carte when composing workflows from a natural-language request. They belong to the CM1 pipeline's end-to-end story. See [`agents/closed-loop-cm1/pipeline-overview.md`](../../agents/closed-loop-cm1/pipeline-overview.md).

## Source

Registration happens in the Flow backend (`akd-services`) at startup. The authoritative list is in `src/api/runtime/app/main.py` of [NASA-IMPACT/akd-services](https://github.com/NASA-IMPACT/akd-services).
