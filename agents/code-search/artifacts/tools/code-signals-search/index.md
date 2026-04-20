---
name: code_signals_search_tool
description: Static code snippet inspection for resolving relevance ambiguity
mcp_server: CMR_MCP_Server
mcp_server_url_env: CODE_SEARCH_MCP_URL
mcp_auth_env: CODE_SEARCH_MCP_KEY
approval: never
affordances: [inspection, mcp]
step: 4
---

# code_signals_search_tool

Inspect code snippets from candidate repositories to resolve relevance ambiguity when README and documentation evidence is insufficient.

## Purpose

Step 4 of the reasoning strategy: **conditional** deep inspection. Used only when README and SDE context aren't enough to determine whether a candidate repository aligns with the user's task.

## When the agent uses it

Only when:

- Step 2 and Step 3 have produced a candidate whose relevance to the user's task is genuinely ambiguous from README / documentation alone.
- The agent needs to look at code-level signals (e.g., does this repo actually implement a specific numerical method?).

The agent references file paths or functions in its output — not full code excerpts.

## Conceptual inputs

- One or more repository URLs and/or code-feature queries (e.g., function names, patterns).

## Conceptual outputs

A set of signals describing code-level evidence:

- Referenced file paths / function names
- Semantic relevance scores or pattern matches
- Description of what was inspected (at a high level, not excerpted)

## Rules the agent follows

- Use sparingly — this tool is conditional and consumes context budget.
- Reference file paths or functions in the output; never embed full code excerpts.
- Record provenance appropriately (the candidate was retained via a combination of sources; note that code-signals clarified relevance).

## Upstream

Backed by `akd_ext/tools/code_search/code_signals.py`. Exposed via the `CMR_MCP_Server` hosted MCP.
