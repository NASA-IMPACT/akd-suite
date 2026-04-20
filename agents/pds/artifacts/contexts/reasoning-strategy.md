# Reasoning Strategy

## Search Rules

### 1. Do not invent facts

You may apply minimal retrieval-oriented normalization, such as expanding common mission or instrument aliases or standardizing target names. If you do, state it explicitly.

### 2. Search at the correct granularity

- First decide whether the request is primarily about:
  - bundles, volumes, collections, or datasets
  - specific observations, granules, or products
- Granularity determines what kind of entity to return, but not the initial routing step.

### 3. Use catalog-first routing for both dataset-level and product-level searches

- If the user is looking for bundles, volumes, collections, datasets, observations, granules, or products, first search with broad catalog-style discovery tools:
  - `PDS_CATALOG_MCP`
  - `PDS4_MCP`
- Use these tools first to identify the best matching candidate datasets, collections, bundles, product groups, or product families.
- During broad catalog-first discovery, explicitly check for both PDS4 and PDS3 representations when available, rather than stopping after the first matching version.
- After identifying strong candidates, narrow with node-specific tools only when needed to:
  - refine results
  - retrieve more specific product-level matches
  - confirm node-specific metadata
  - obtain stable product pages, endpoints, or download locations

### 4. Use node-specific tools as a narrowing or follow-up step

- After catalog-first discovery, narrow using the mapped node/service when appropriate:
  - GEO → `ODE_MCP`
  - IMG → `IMG_MCP`
  - RMS → `OPUS_MCP`
  - SBN → `SBN_MCP`
  - PPI / ATM → usually remain in `PDS4_MCP` or `PDS_CATALOG_MCP` unless a node-specific follow-up is clearly needed
- Do not begin with node-specific tools unless catalog-first discovery is impossible or the user explicitly requires a known node/service workflow.

### 5. Broad-first is the default for all discovery-style queries

Including dataset-level and product-level requests.

- Start with `PDS_CATALOG_MCP` and/or `PDS4_MCP`.
- Then narrow with filters or node-specific tools as needed.
- If a search returns no useful results, relax constraints rather than stacking more filters.

### 6. Exact identifiers are a special case

- If the user provides an exact dataset ID, LID, LIDVID, PRODUCT_ID, OPUS_ID, or ODE_ID, you may go directly to the most appropriate resolving tool.
- Even in this case, use only the minimal additional calls needed to confirm metadata, parent context, or stable access paths.
- If relevant, still check whether a corresponding PDS4 or PDS3 counterpart exists.

### 7. Version preference and cross-version coverage

- When relevant data exists in both PDS4 and PDS3 forms, return both.
- Prefer PDS4 first in ranking and presentation, but also include the corresponding PDS3 version if available.
- Do not stop after finding only one version.
- Clearly label each result as PDS4 or PDS3.
- Describe cross-version relationships only when supported by identifiers, titles, descriptions, archive lineage, or node metadata.
- If the relationship is uncertain, mark it as `likely_related` or `unknown` rather than assuming equivalence.
- When a matching PDS3 result is found, also check whether a corresponding PDS4 version, migration, successor collection, or equivalent product family is available.
- When a matching PDS4 result is found, also check whether a corresponding legacy PDS3 version exists when it is still relevant for discovery or comparison.

### 8. Stop when you have a strong answer

- If a dataset, collection, or product clearly matches the user's query, stop broad exploration.
- Make only the minimal extra calls needed to complete required metadata, parent context, or one representative lower-level example if relevant.
- Do not keep searching just to pad the number of results.

### 9. Avoid search loops

- If repeated searches with the same tool are not improving results, switch tool type or return best partial results.
- Do not re-fetch an entity already confirmed unless needed to fill required metadata.

### 10. Allow partial success

- If some facets succeed and others fail, return the successful results and clearly label unresolved parts.
- Use a hard stop only if the whole request cannot be searched responsibly.

## Default Workflow

Interpret → Clarify only if needed → Choose granularity → Search broad first with `PDS_CATALOG_MCP` / `PDS4_MCP` → Check for both PDS4 and PDS3 representations when available → Narrow with node-specific tools if needed → Execute bounded searches → Collect candidates → Dedupe → Attach one parent level up when available → Return results.
