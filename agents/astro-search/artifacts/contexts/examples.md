# Example Interactions

These examples ship in the agent's system prompt and illustrate how each search pattern manifests in practice.

## Example 1: Object-based search

**User:** "Find X-ray observations of NGC 1275"

**Agent actions:**

- Resolve "NGC 1275" via SIMBAD → get coordinates
- Search HEASARC for X-ray observations at those coordinates
- Present Chandra, XMM-Newton, etc. observations found

## Example 2: Literature-driven

**User:** "I'm studying tidal disruption events. What data is available?"

**Agent actions:**

- Ask: "Are you looking for data on a specific TDE, or surveying available observations across known TDEs? Any wavelength preference?"
- Based on answer, either search ADS for TDE papers with data links, or search archives for known TDE positions

## Example 3: Event follow-up

**User:** "What observations exist around GW170817?"

**Agent actions:**

- Ask: "What time window around the merger? And what wavelength bands are you interested in?"
- With T0 and time window, search MAST/HEASARC for observations overlapping that period
- Prioritize by temporal proximity to T0

## Example 4: Specific archive query

**User:** "Show me all TESS observations of WASP-121"

**Agent actions:**

- Resolve "WASP-121" via SIMBAD
- Search MAST specifically for TESS data at those coordinates
- Return TESS sectors, cadence, data products available
