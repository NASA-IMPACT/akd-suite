# Additional Context — CMR Search API Reference

This reference material is embedded in the agent's system prompt as additional context, so the agent can ground its tool calls in specific CMR endpoints, parameters, and response shapes.

## Overview

The Common Metadata Repository (CMR) Search API provides access to NASA Earth science metadata, enabling programmatic discovery and retrieval of collections, granules, and related concepts. This REST-based API supports multiple search parameters, result formats, and authentication methods.

**Base URL**: `https://cmr.earthdata.nasa.gov/search/`

## Key Concepts & Terminology

### Core CMR Concepts

- **Collection**: A grouping of related data files or granules, representing a dataset
- **Granule**: Individual data files within a collection (e.g., a single HDF file)
- **Concept ID**: Unique identifier for CMR concepts in format `<concept-type-prefix><unique-number>-<provider-id>`
  - Collection concept IDs start with "C" (e.g., `C123456-LPDAAC_ECS`)
  - Granule concept IDs start with "G" (e.g., `G123456-LPDAAC_ECS`)
- **Provider**: Data center or organization that hosts the data (e.g., LPDAAC_ECS, NSIDC_ECS)
- **Instrument**: Sensor that collected the data (e.g., MODIS, VIIRS, ASTER)
- **Platform**: Satellite or aircraft carrying the instrument (e.g., Terra, Aqua, Landsat-8)
- **Dataset**: Another term for collection, representing a coherent set of data

### Metadata Standards

- **UMM** (Unified Metadata Model): NASA's standard for Earth science metadata
- **ECHO**: Legacy metadata format (Earth Observing System Clearinghouse)
- **DIF** (Directory Interchange Format): GCMD metadata format
- **STAC** (Spatio-Temporal Asset Catalog): Modern geospatial metadata standard

## API Endpoints

### Primary Search Endpoints

- **Collections**: `/search/collections` — Search for datasets/collections
- **Granules**: `/search/granules` — Search for individual data files
- **Variables**: `/search/variables` — Search for science variables
- **Services**: `/search/services` — Search for data services
- **Tools**: `/search/tools` — Search for analysis tools

### Utility Endpoints

- **Autocomplete**: `/search/autocomplete` — Get search suggestions
- **Facets**: Access faceted search capabilities

## Authentication

### Token Types

1. **EDL Bearer Token**: Earth Data Login token
2. **Launchpad Token**: Legacy authentication system

### Authentication Methods

- **Authorization Header**: `Authorization: Bearer <token>`
- **Token Parameter**: `?token=<token>` in URL

### Example

```bash
# Using Authorization header
curl -H "Authorization: Bearer YOUR_TOKEN" "https://cmr.earthdata.nasa.gov/search/collections"

# Using token parameter
curl "https://cmr.earthdata.nasa.gov/search/collections?token=YOUR_TOKEN"
```

## Request Parameters

### Common Parameters

- `page_size`: Number of results per page (default: 10, max: 2000)
- `page_num`: Page number to return (1-based)
- `sort_key`: Field(s) to sort results by
- `concept_id`: Search by unique identifier
- `provider`: Filter by data provider
- `token`: Authentication token

### Collection Search Parameters

- `keyword`: Text search across collection metadata
- `short_name`: Collection short name
- `version`: Collection version
- `temporal`: Temporal range in format `YYYY-MM-DDTHH:mm:ssZ,YYYY-MM-DDTHH:mm:ssZ`
- `platform`: Platform/satellite name
- `instrument`: Instrument name
- `science_keywords`: Science keyword hierarchy
- `project`: Project or mission name
- `processing_level`: Data processing level (L0, L1A, L1B, L2, L3, L4)
- `data_center`: Data center name
- `archive_center`: Archive center name
- `spatial`: Spatial search parameters

### Granule Search Parameters

- `collection_concept_id`: Filter granules by collection
- `temporal`: Temporal range for granule search
- `bounding_box`: Spatial bounding box `[west,south,east,north]`
- `point`: Point search `[longitude,latitude]`
- `polygon`: Polygon search (WKT format)
- `producer_granule_id`: Producer-assigned granule ID
- `online_only`: Return only online-accessible granules
- `downloadable`: Return only downloadable granules
- `cloud_cover`: Cloud cover percentage range

### Advanced Parameters

- `options[case_sensitive]`: Case-sensitive search (true/false)
- `options[pattern]`: Enable pattern matching (true/false)
- `options[ignore_case]`: Ignore case in search (true/false)
- `options[and]`: AND logic for multiple values (true/false)

## Response Formats

### Supported Formats

- **JSON**: Default format, comprehensive metadata
- **XML**: Various XML schemas available
- **ATOM**: XML feed format
- **CSV**: Comma-separated values
- **KML**: Keyhole Markup Language for mapping
- **STAC**: Spatio-Temporal Asset Catalog
- **UMM JSON**: Unified Metadata Model JSON

### Format Selection

- **Accept Header**: `Accept: application/json`
- **Extension**: `.json`, `.xml`, `.atom`, `.csv`, `.kml`, `.stac`
- **Format Parameter**: `?format=json`

### Example Response Structure (JSON)

```json
{
  "hits": 1234,
  "took": 45,
  "items": [
    {
      "concept_id": "C123456-LPDAAC_ECS",
      "revision_id": 1,
      "provider_id": "LPDAAC_ECS",
      "short_name": "MOD09A1",
      "version_id": "6.1",
      "meta": {
        "concept_type": "collection",
        "native_id": "MOD09A1_V6.1",
        "provider_id": "LPDAAC_ECS"
      },
      "umm": {
        "EntryTitle": "MODIS/Terra Surface Reflectance 8-Day L3 Global 500m SIN Grid V061",
        "ShortName": "MOD09A1",
        "Version": "6.1",
        "DataDates": [
          { "Date": "2000-02-18T00:00:00.000Z", "Type": "CREATE" }
        ],
        "Platforms": [
          { "ShortName": "Terra", "LongName": "Earth Observing System, Terra" }
        ]
      }
    }
  ]
}
```

## Rate Limiting & Performance

### Limits

- **Request Timeout**: 180 seconds maximum
- **Query Timeout**: 170 seconds internal timeout
- **Rate Limiting**: 429 status code with `retry-after` header
- **URL Length**: ~6,000 characters maximum

### Optimization Tips

- Use `page_size` for pagination instead of large single requests
- Implement exponential backoff for rate limit responses
- Use specific search parameters to reduce result sets
- Consider using scroll/search-after for large result sets

## Error Handling

### Common HTTP Status Codes

- **200**: Success
- **400**: Bad Request (invalid parameters)
- **401**: Unauthorized (authentication required)
- **403**: Forbidden (insufficient permissions)
- **404**: Not Found
- **429**: Too Many Requests (rate limited)
- **500**: Internal Server Error

### Error Response Format

```json
{
  "errors": [
    {
      "code": "INVALID_PARAMETER",
      "message": "Parameter 'temporal' is not valid"
    }
  ]
}
```

## Common Usage Patterns

### 1. Find Collections by Keyword

```bash
GET /search/collections?keyword=temperature&provider=NSIDC_ECS
```

### 2. Get Granules for a Specific Collection

```bash
GET /search/granules?collection_concept_id=C123456-LPDAAC_ECS&temporal=2023-01-01T00:00:00Z,2023-12-31T23:59:59Z
```

### 3. Spatial Search

```bash
GET /search/collections?bounding_box=-180,-90,180,90&platform=Terra
```

### 4. Paginated Results

```bash
GET /search/collections?page_size=50&page_num=2&sort_key=short_name
```

## Integration Notes for MCP Server

### Typical Workflow

1. **Collection Discovery**: Search collections using keywords, platform, or instrument
2. **Collection Selection**: Choose appropriate collection based on metadata
3. **Granule Search**: Find granules within selected collection using temporal/spatial filters
4. **Data Access**: Use granule metadata to access actual data files

### Key Fields for MCP Integration

- **Collection**: `concept_id`, `short_name`, `version_id`, `entry_title`
- **Granule**: `concept_id`, `producer_granule_id`, `online_access_urls`
- **Temporal**: `temporal_extent`, `temporal`
- **Spatial**: `bounding_box`, `polygons`

### Authentication Considerations

- EDL tokens are preferred for new integrations
- Tokens should be securely stored and refreshed as needed
- Consider implementing token validation before API calls

## Additional Resources

- **CMR Documentation**: https://cmr.earthdata.nasa.gov/search/site/docs/search/api.html
- **UMM Specification**: https://earthdata.nasa.gov/eosdis/science-system-description/eosdis-components/common-metadata-repository
