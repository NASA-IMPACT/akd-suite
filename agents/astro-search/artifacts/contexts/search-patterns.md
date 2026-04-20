# Search Approach Patterns

Recognize which pattern fits the user's request:

## A) Literature-Driven

User starts from papers, topics, or science questions.

- Search ADS for relevant papers using keywords
- Extract object names, instruments, and observational context from abstracts/metadata
- Resolve objects via SIMBAD to get coordinates
- Search archives based on extracted context

**Special case — Objects from bibcode:**

- Use `simbad_query_bibobj` to get all objects mentioned in a paper
- Returns objects with coordinates and `obj_freq` (mention count — higher = more central to paper)
- Then use coordinates to query archives for each object

## B) Coordinate-Driven

User provides coordinates or a sky region.

- Validate coordinates (assume ICRS unless stated otherwise)
- Search archives using cone search with specified radius
- Cross-match across catalogs if multiple wavelengths needed

## C) Archive-Driven

User wants data from specific missions/instruments.

- Query the specified archive directly (MAST, HEASARC, IRSA)
- Apply user constraints (time window, instrument mode, calibration level)
- Return available observations matching criteria

## D) Event/Alert-Driven

User is following up on a transient event (GW, GRB, neutrino alert).

- Requires: event time (T0), time window (±Δt), and spatial region
- Search for observations overlapping the event window and localization
- Prioritize by temporal proximity to T0 and data readiness

## Minimum Required Information

Before searching, ensure you have the minimum information for the search pattern:

**For object-based searches:**

- Object name OR coordinates
- Data type needed (image, spectrum, lightcurve, catalog) OR wavelength/energy band

**For coordinate searches:**

- RA/Dec (ICRS)
- Search radius
- Data type OR wavelength band

**For event follow-up:**

- Event reference (ID, time, or localization)
- Time window around event
- Spatial region (coordinates + radius, or polygon vertices)

**For literature-driven:**

- Keywords, topic, or science question

If any required information is missing, ask the user before proceeding. Do not guess critical parameters like:

- Observation times or time windows
- Exposure requirements
- Calibration levels
- Proprietary dates
