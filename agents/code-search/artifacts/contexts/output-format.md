# Output Format

## Authoritative Output (Mandatory)

A single deterministic JSON object conforming to the fixed schema, containing:

- Zero to six repositories.
- For each repository:

```json
{
  "name": "string",
  "url": "string -- primary URL: the code site URL that the ASCL entry links to, or the GitHub/GitLab repo, or the official project website",
  "url2": "string or null -- secondary URL: if available, include a second relevant URL from a different source (e.g., if url is the ASCL-linked project site, url2 could be the GitHub repo or vice versa). Set to null if only one URL was found.",
  "ranking_position": "integer (1-6)",
  "rationale_for_inclusion": "string -- reason for inclusion, including what discovery channel found it",
  "fit_notes_and_limitations": "string -- alignment notes and limitations for the user's specific task",
  "provenance": "string -- tool/source used for discovery (e.g., 'NASA Repository Search', 'ADS', 'SDE Text Search', 'External Web Search')",
  "ads_evidence": {
    "bibcodes": ["list of ADS bibcodes -- the FIRST entry should be the code's canonical paper (the 'Described in' bibcode from the ASCL record when available), followed by papers that use the code for the queried task"],
    "citation_count": "integer -- number of ADS papers found referencing this repository",
    "usage_summary": "string or null -- brief description of how the code was used in literature, with specific mention of relevance to the user's queried task"
  }
}
```

Note: The `ads_evidence` object is populated only for Astrophysics queries; for all other domains, return null or empty values for these fields.

## Excluded Candidates (Mandatory when applicable)

If any candidates were found during the search process but excluded from the final ranked list, append:

```json
{
  "excluded_candidates": [
    {
      "name": "string",
      "reason_for_exclusion": "string -- why this candidate was not included in the final list"
    }
  ],
  "unlocated_known_codes": [
    {
      "name": "string",
      "note": "string -- why this well-known code could not be found through available channels"
    }
  ]
}
```

## Optional Companion Output

A human-readable narrative and/or table:

- MUST be semantically equivalent to the JSON.
- MUST introduce no new information.
- Should highlight ADS citation findings as part of the comparative discussion.
- Should explicitly mention any well-known codes that were searched for but could not be located.
