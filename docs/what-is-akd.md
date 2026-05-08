# What is AKD?

**Accelerated Knowledge Discovery (AKD)** is a NASA-IMPACT program that builds AI-augmented tools to accelerate how scientists find, evaluate, and reason about scientific knowledge — datasets, code, literature, methods — across NASA's Science Mission Directorate (SMD) divisions:

- **Earth Science** — atmosphere, ocean, land, cryosphere, biosphere, solid Earth
- **Planetary Science** — Mars, outer planets, small bodies, planetary atmospheres
- **Astrophysics** — cosmology, stars, galaxies, high-energy astrophysics
- **Heliophysics** — the Sun and its influence on the solar system
- **Biological and Physical Sciences** — microgravity, life sciences, physical sciences in space

## Why a program, not a product

Scientific workflows are long, specialized, and full of hand-offs: literature searches, dataset discovery, code selection, feasibility analysis, experiment design, report drafting. Each of those steps has its own vocabulary, its own trusted sources, and its own authoritative metadata systems. A one-size-fits-all chatbot doesn't help.

AKD's approach is a **composable multi-agent framework** where each step of a scientific workflow is addressed by a purpose-built agent that knows its data source, reasoning strategy, and guardrails. Those agents plug into higher-level workflows. Humans stay in the loop for approval and interpretation.

## What exists today

- **Domain agents** for data discovery (CMR, PDS, Astro), code discovery, and research gap analysis — each with a documented reasoning strategy and dedicated tool set.
- **A five-stage closed-loop pipeline (CM1)** — an end-to-end research workflow that takes a research question through gap analysis, capability/feasibility mapping, workflow-spec building, experiment implementation, and research-report generation, with human approval gates between stages.
- **A framework-level guardrail system** with Granite Guardian (content moderation) and a Risk Agent (science-risk LLM judge) composable over any agent's input or output.
- **A web product (Flow)** that exposes the agents through a natural-language planner and a visual workflow canvas. Deployed at [flow.akd.odsi.io](https://flow.akd.odsi.io).
- **A playground (Labs)** for building and benchmarking experimental agents. Deployed at [labs.akd.odsi.io](https://labs.akd.odsi.io).

## Who AKD is for

- **Working scientists** who need to find datasets, code, or literature faster than manual searching allows, and want reproducible, traceable results.
- **Research software engineers** who want to build new domain agents without re-inventing streaming, human-in-the-loop, tool orchestration, and safety primitives.
- **Mission and division leads** who need visibility into how agents are reasoning and where their outputs come from.

## What AKD is *not*

- **Not an autonomous scientific decision-maker.** Every agent in AKD is designed to be non-prescriptive: it surfaces candidates, evidence, and caveats, but a human researcher retains authority over selection, interpretation, and publication.
- **Not a replacement for the authoritative sources** (CMR, PDS, ADS, SDE, etc.). AKD agents query those systems and organize results; they don't fork or re-host the underlying data.

## Next steps

- Read [`ecosystem.md`](./ecosystem.md) for how the five AKD repos fit together.
- See the [`agents/`](../agents/) catalog for what each published agent does.
- See [`flow/user-guide.md`](../flow/user-guide.md) if you want to try it.
