# Special Cases

## Objects from Bibcode

When user requests objects mentioned in a paper by bibcode:

**Tool:** `simbad_query_bibobj`
**Parameters:** `{"bibcode": "YYYY.JOURNAL..PAGE.A"}`
**Returns:** Table with:

- `main_id`: Primary object identifier
- `ra`, `dec`: Coordinates (ICRS)
- `bibcode`: Citation reference
- `obj_freq`: Number of times object mentioned in paper (higher = more central to paper)

**Example Output:**

| main_id | ra | dec | obj_freq |
| --- | --- | --- | --- |
| IC 4997 | 305.036 | -16.836 | 44 |
| BD+33 2642 | 237.999 | 33.677 | 3 |

Objects with higher `obj_freq` are more central to the paper's study.

## Data Product URLs

When user requests data product URLs (FTP/download links):

**Current limitation:** `locate_data` function not directly exposed in MCP tools.

**Workaround:**

1. Query catalog (e.g., `heasarc_query_region` with `catalog="chanmaster"`)
2. **CRITICAL** — Filter results before extracting URLs:
   - `exposure > 0` (exclude non-observations)
   - `grating == "HETG"` (for specific instrument)
   - `time_stop < "2023-01-01"` (for date constraints)
3. Extract URLs from query results:
   - Look for `datalink`, `data_url`, or `access_url` columns
   - Or construct from ObsID pattern if known

**Example for Chandra:**

- Query: `heasarc_query_region(catalog="chanmaster", position=coords)`
- Filter: Keep rows where `grating=="HETG"` and `exposure>0`
- URLs follow pattern: `https://heasarc.gsfc.nasa.gov/FTP/chandra/data/byobsid/{last_digit}/{obsid}/`
- Where `obsid` from `obsid` column, `last_digit = obsid[-1]`

**Alternative:** Use `mast_get_product_list` for MAST missions (returns `data_uri` that can be downloaded).
