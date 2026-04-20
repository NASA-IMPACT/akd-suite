# Labs — Agent Development Lifecycle

How an experimental agent goes from "initial idea" to "ready to publish to Flow" inside Labs.

## The loop

```
   ┌──────────────┐
   │ 1. Create    │   Author system prompt, pick model, configure tools
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │ 2. Chat-test │   Interact in the Chat lab; observe real responses
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │ 3. Read      │   Inspect full trace: system prompt, tool calls,
   │    trace     │   reasoning, tokens, cost
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │ 4. Iterate   │   Refine prompt, swap tools, change model settings
   └──────┬───────┘
          │
          │  (loop back to step 2)
          │
          ▼
   ┌──────────────┐
   │ 5. Benchmark │   Run against a prepared test suite
   └──────┬───────┘
          │
          │  (loop back to step 4 if needed)
          │
          ▼
   ┌──────────────┐
   │ 6. Publish   │   Within Labs: toggle visibility to project/org
   │    in Labs   │
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │ 7. Publish   │   Outside Labs: add to akd-ext, redeploy Flow
   │    to Flow   │   (see publishing-to-flow.md)
   └──────────────┘
```

## Guidance for each stage

### 1. Create

Start with a clear, narrow role: what is this agent for, what data sources will it use, what is out of scope? If you're building in a NASA-SMD domain (Earth, Planetary, Astrophysics, Heliophysics, BPS), look at [`agents/`](../agents/) for patterns — the CARE-style prompts used by CMR, PDS, and Code Search are a good template.

Tool config: prefer hosted MCP servers over in-process tools for anything that needs external data. You can reference the same MCP servers the published Flow agents use (CMR, PDS, ADS, Astroquery, etc.) if you have the URL env vars configured.

### 2. Chat-test

Exercise the agent against prompts representative of your user base. Especially test edge cases:

- Off-scope requests — does the agent refuse cleanly?
- Ambiguous requests — does it clarify?
- Tool failures — does it degrade gracefully?

### 3. Read trace

Traces are where Labs really earns its keep. For each interesting turn:

- Look at exactly which tools were called, with what arguments.
- Check if the model reasoning summary suggests the agent understood the task.
- Notice token counts. A balloon in tokens often means the agent is ingesting too much context — consider pruning or caching.
- Note the cost estimate. Multiply by expected usage volume for a real cost projection.

### 4. Iterate

Most iteration happens at the prompt level:

- **Add a PROCESS section** if the agent skips steps.
- **Tighten CONSTRAINTS & STYLE RULES** if it says things it shouldn't.
- **Specify OUTPUT FORMAT** more precisely if downstream consumers need structure.
- **Add WHEN-STUCK handling** if it fails silently.

Swap models / reasoning efforts to trade off cost vs. quality.

### 5. Benchmark

Once chat-testing stabilizes, move to benchmarks. A benchmark suite lets you compare configurations objectively — A/B two prompt versions, or measure cost vs quality across models.

### 6. Publish in Labs

Set `visibility_scope` appropriately — `project` if it's only useful to a single team, `organization` if the whole org should discover it. `published_by_user_id` records the promotion for audit.

### 7. Publish to Flow

Currently, the Labs → Flow path is **not a Labs feature** — it requires modifying `akd-ext` and redeploying the Flow backend. See [`publishing-to-flow.md`](./publishing-to-flow.md) for both the current mechanism and the planned future artifact-based workflow.

## Where to read next

- [`user-guide.md`](./user-guide.md) — the UI walkthrough.
- [`benchmarking.md`](./benchmarking.md) — how to structure benchmark suites.
- [`publishing-to-flow.md`](./publishing-to-flow.md) — current + planned promotion path.
- [`../frameworks/akd-ext/build-a-custom-agent.md`](../frameworks/akd-ext/build-a-custom-agent.md) — what a Labs-validated prompt looks like as a published akd-ext agent.
