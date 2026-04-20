# Search Workflow

1. **Parse the request** — Identify what the user wants (data type, wavelength, target, time constraints)

2. **Check for missing information** — If minimum requirements aren't met, ask focused questions:
   - Priority 1: What type of data? (images, spectra, lightcurves, catalogs)
   - Priority 2: What wavelength/energy band? Or which mission/instrument?
   - Priority 3: Any time constraints? What spatial region/radius?

3. **Resolve identity** — If object name given, resolve via SIMBAD
   - If multiple matches, ask user to choose
   - Report coordinates you'll use for archive searches

4. **Search archives** — Use appropriate MCP tools based on the science case:
   - High-energy → `heasarc_query_region`
   - UV/Optical space → `mast_query_region`
   - Infrared → `irsa_query_region`
   - Extragalactic → `ned_query_region`
   - Astrometry → `gaia_cone_search`
   - Literature → `ads_query_simple`
   - Objects from paper → `simbad_query_bibobj`
   - If unsure, start with one archive; offer to expand if results are limited

5. **Execute MCP tool calls with error handling** — Check for:
   - Empty results (explain what was searched, suggest alternatives)
   - Missing coordinates (ask user to clarify)
   - Tool errors (report error, try alternative approach)

6. **Present results** — For each candidate dataset, provide:
   - Archive and mission/instrument
   - Observation ID
   - Time range (if available)
   - Exposure time (if available)
   - Calibration/processing level (if available)
   - Access URL or how to retrieve
   - Any caveats (proprietary status, quality flags)

7. **Iterate** — If results are sparse or not what user expected:
   - Offer to search additional archives (with user approval)
   - Suggest relaxing constraints (ask user which to relax first)
   - Try alternative search strategies
