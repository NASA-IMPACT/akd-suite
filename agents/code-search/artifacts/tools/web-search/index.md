---
name: web_search
description: OpenAI Agents SDK WebSearchTool for supplementary web search
mcp_server: null
approval: default
affordances: [search, web]
step: 6
---

# WebSearchTool (supplementary)

The OpenAI Agents SDK's built-in `WebSearchTool`, used only in Step 6 of the reasoning strategy for supplementary web search.

## Purpose

Step 6 of the reasoning strategy: **Completeness Check & Supplementary Web Search**. The primary use case is finding public repositories for codes on the Expected Codes checklist that earlier steps (NASA repo search, SDE, ADS) did not surface.

## When the agent uses it

Only in Step 6. Before ranking (Step 7), the agent compares the running candidate list against the Expected Codes checklist one more time, identifies missing codes, and uses web search to locate their public repositories.

## Conceptual inputs

A single consolidated web search query per call. **All missing code names combined into one query** (e.g., "FLASH GitHub" "PLUTO GitHub" "Enzo GitHub") rather than separate queries per code.

## Conceptual outputs

Web search result set (title, snippet, URL).

## Rules the agent follows

- **Maximum 3 web search queries per run.** Aim to resolve all missing codes in 1–2 queries.
- Do not re-query earlier tools in Step 6 — only web search is available.
- Do not spend multiple queries on a single code — if the first search doesn't find a code's public repo, move on and note it as unlocated.
- Prioritize `.gov`, `.edu`, `nasa.gov`, `esa.int`, and similar trusted domains.
- **Explicitly flag** all externally sourced repositories in the `provenance` field of the output.
- If a well-known code from the Expected Codes checklist cannot be found through web search, include it in `unlocated_known_codes` with a note; do not silently drop it.

## Upstream

Provided by the OpenAI Agents SDK; no MCP server is involved.
