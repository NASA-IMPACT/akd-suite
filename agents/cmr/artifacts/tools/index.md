# CMR Agent — Tools

The CMR CARE Agent wires a single hosted MCP server, `CMR_MCP_Server`, with a whitelist of three tools. All three tools are executed server-side by the MCP host; the agent sees typed inputs and outputs.

## MCP server

- **Label**: `CMR_MCP_Server`
- **URL env var**: `CMR_MCP_URL`
- **Approval gating**: never (the agent is trusted to invoke these without per-call user approval)
- **Server description**: CMR MCP server for NASA dataset discovery

## Allowed tools

- [`search-collections/`](./search-collections/) — search the CMR collection index.
- [`get-granules/`](./get-granules/) — retrieve granules for a selected collection.
- [`get-collection-metadata/`](./get-collection-metadata/) — fetch rich metadata for a specific collection.

## Notes

- The MCP server handles authentication, rate limiting, and response shaping against `cmr.earthdata.nasa.gov`. Agent callers do not need EDL tokens themselves.
- Tool names match exactly what the agent's `allowed_tools` list whitelists in its config. Any tool not in this list will not be called, even if the MCP server advertises it.
- See [`contexts/cmr-api-reference.md`](../contexts/cmr-api-reference.md) for the underlying CMR Search API these tools wrap.
