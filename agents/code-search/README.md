# Code Search Agent

The **Code Search CARE Agent** is an AKD agent for discovering publicly available scientific code repositories that plausibly align with a user's technical or scientific task. It follows the **CARE** reasoning loop and operates as a **read-only, decision-support system** — non-prescriptive, non-endorsing, and human-in-the-loop by design.

## At a glance

| | |
| --- | --- |
| **Purpose** | Identify and comparatively describe public scientific code repositories matching a user's task; never recommend or endorse |
| **Users** | Scientists and research software engineers across NASA SMD domains (Astrophysics, Earth, Heliophysics, Planetary, BPS) |
| **Data sources** | NASA-verified repository corpus, Science Discovery Engine (SDE), NASA ADS, code snippet inspection, supplementary web search |
| **Reasoning pattern** | Multi-pass discovery: intent parse → NASA corpus → SDE → deep inspection → ADS (Astrophysics only) → web supplement → rank → disclose |
| **Output** | Deterministic JSON with up to 6 ranked repositories, excluded candidates, and unlocated known codes |
| **Source** | [NASA-IMPACT/akd-ext — `akd_ext/agents/code_search_care.py`](https://github.com/NASA-IMPACT/akd-ext) |

## What a successful run looks like

The user describes a technical or scientific task. The agent:

1. Parses intent, extracts constraints, categorizes the domain (Astrophysics, BPS, Earth, Heliophysics, Planetary), and generates an Expected Codes checklist from its domain knowledge.
2. Runs **primary discovery** against the NASA-verified repository corpus via multiple targeted queries.
3. **Enriches context** through the Science Discovery Engine (SDE) — especially valuable for Earth/Helio/Planetary queries; lighter for Astrophysics.
4. Optionally performs **deep code inspection** via a snippet search tool when README + SDE context is insufficient.
5. For **Astrophysics queries only**, performs a **literature search in ADS** to discover additional codes that comparison/benchmark/review papers name.
6. Runs a **completeness check** and uses **web search** to locate public repos for codes on the Expected Codes checklist that earlier steps missed.
7. Evaluates and ranks all candidates (max 6) using intent alignment, scientific citation evidence, documentation quality, maintenance, trust, and metadata signals.
8. **Discloses** evidence, uncertainty, conflicting signals, excluded candidates, and well-known codes that couldn't be located.

## What it never does

- Uses prescriptive language: "best," "recommended," "final choice," "approved," "use this"
- Executes, clones, downloads, or tests code
- Treats popularity signals (stars, forks) as decisive
- Fabricates repositories, metadata, or capabilities
- Accesses private, gated, or credential-restricted sources
- Silently drops candidates without explaining why in the output

## Deeper reading

- [`artifacts/`](./artifacts/) — role, constraints, reasoning strategy, output format, and tool-by-tool descriptions.
- Upstream implementation: [`akd_ext/agents/code_search_care.py`](https://github.com/NASA-IMPACT/akd-ext).
- Related: [`agents/cmr/`](../cmr/) and [`agents/astro-search/`](../astro-search/) for sister CARE agents; [`frameworks/akd-ext/`](../../frameworks/akd-ext/) for the `RepositorySearchTool` and `CodeSignalsSearchTool` reference.
