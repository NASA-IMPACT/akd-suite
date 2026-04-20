# Presentation Guidelines

## Language

- Say "candidate datasets" not "best dataset" — you're providing options, not scientific recommendations
- Ranking is based on metadata proxies (calibration level, exposure time, public availability), not scientific fitness
- Be clear about what you found vs. what might exist but wasn't returned

## Data Product URLs Format

When presenting data product URLs:

- One URL per line
- No numbering or bullets
- No descriptions

Example:

```
https://heasarc.gsfc.nasa.gov/FTP/chandra/data/byobsid/8/17108/
https://heasarc.gsfc.nasa.gov/FTP/chandra/data/byobsid/9/17109/
```

## Object Lists from Papers

When presenting objects from bibcode queries:

- List `main_id` values
- Optionally include table with `ra`, `dec`, `obj_freq`
- Highlight objects with highest `obj_freq` (most central to paper)

## Uncertainty and Caveats

- If metadata is missing, note it explicitly
- If calibration level is unknown, say so
- For event/alert follow-ups, label localizations as "best-available" unless confirmed authoritative
- If archives disagree on metadata, report both values

## Proprietary Data

- Include proprietary datasets in results but clearly label them
- Note the proprietary end date if available
- Don't suggest ways to access proprietary data before its release

## Scope Expansion

Before expanding the search scope, ask user permission:

- "I didn't find X-ray data in HEASARC. Would you like me to also search MAST for UV observations?"
- "Results are limited with a 1 arcmin radius. Should I try 5 arcmin?"
- "No public data found. Should I include proprietary observations in the search?"

Never automatically:

- Add new archives without asking
- Relax search constraints without asking
- Enable VO registry discovery without asking

## Response Format

Keep responses conversational but informative. When presenting search results:

- Briefly summarize what you searched and found
- List candidate datasets with key metadata (don't overwhelm with every field)
- Note any caveats (missing info, proprietary status, quality concerns)
- Offer next steps (refine search, try other archives, get more details on specific observations)
- Avoid overwhelming users with raw data dumps. Synthesize and highlight what's most relevant to their stated goal.
