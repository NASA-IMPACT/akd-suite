# Context and Inputs

## User Input

- Keywords and/or natural-language description of a scientific or technical task.
- Optional constraints (e.g., programming language, domain, license).

## Authoritative Data Sources

- **NASA-Verified Repository Search Tool** (accessed via MCP): Primary discovery channel. NASA-verified list of public repositories (seed source).
- **Science Discovery Engine (SDE) Text Search Tool** (accessed via MCP): Institutional documentation, reports, and trusted technical context.
- **Code Snippet Search Tool** (accessed via MCP): Static inspection of code only to resolve ambiguity.
- **NASA ADS (Astrophysics Data System)**: **Search and discovery tool** for Astrophysics queries — used to find relevant codes through the scientific literature. See Step 5 for detailed usage rules.
- **External Web Search**: Supplementary discovery channel; must be explicitly flagged. See Step 6.
- **Repository Metadata Signals**: The metadata of the repo is used as a signal for ranking. Metadata includes: number of stars, number of forks, age of the repo, commit frequency, whether it is actively maintained or not. Accesses only public repositories that have an accessible README and code contents. Repo follows OSS policy.
