# Tools and Archives

## Object Resolution

**SIMBAD:** Resolve object names to coordinates, get canonical names, aliases, object types.

- Tools: `simbad_query_object`, `simbad_query_region`, `simbad_query_bibobj`
- If SIMBAD returns multiple matches, present options and ask user to confirm.
- `simbad_query_bibobj`: Get all objects mentioned in a paper by bibcode (returns `main_id`, `ra`, `dec`, `obj_freq`).

## Literature Search

**NASA ADS:** Search papers by keywords, author, title, abstract.

- Tool: `ads_query_simple`
- Available fields: bibcode, doi, title, author, abstract, pubdate, citation_count, data links
- Can retrieve references and citations for a given paper

## NASA Archives (Primary)

**MAST:** HST, TESS, Kepler, JWST, GALEX, and other UV/optical space missions.

- Tools: `mast_query_object`, `mast_query_region`, `mast_get_product_list`, `mast_download_products`

**HEASARC:** High-energy missions (Chandra, XMM, Swift, Fermi, NICER, NuSTAR, etc.).

- Tools: `heasarc_query_region`, `heasarc_download_data`, `heasarc_query_tap`

**IRSA:** Infrared surveys and missions (Spitzer, WISE, 2MASS, IRAS, etc.).

- Tools: `irsa_query_region`

## Other Services

- **NED:** `ned_query_region` for extragalactic objects
- **Gaia:** `gaia_cone_search`, `gaia_query_object` for astrometry
- **VizieR:** `vizier_query_region` for catalog cross-matching

## VO Services

- **TAP/ADQL:** For complex catalog queries
- **SIA:** Simple Image Access for finding images
- **SSA:** Simple Spectral Access for finding spectra
