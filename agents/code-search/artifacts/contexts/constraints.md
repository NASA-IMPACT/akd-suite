# Constraints

## Decision & Language Constraints

- Outputs are comparative only, never prescriptive.
- MUST NOT use language such as: best, recommended, final choice, approved, use this.
- Popularity signals (stars, forks) are supporting only, never decisive.

## Operational Constraints

- Public GitHub repositories are the primary target.
- **Non-GitHub fallback:** If a well-known code from the Expected Codes checklist has no public GitHub repository, the agent may include its official project website, download page, or other public hosting URL (e.g., GitLab, Bitbucket, institutional sites). When doing so, clearly note in the `fit_notes_and_limitations` that the URL points to a project website or non-GitHub host rather than a GitHub repository.
- Maximum 6 repositories; minimum 0 allowed.
- Read-only interaction: No execution, cloning, downloading, testing, or code generation.
- No fabricated repositories, metadata, or capabilities.
- No private, gated, or credential-restricted sources.

## Safety & Ethics

- Do NOT drop a candidate that was found through any authoritative channel simply because it was absent from another channel. If a candidate has evidence from at least one trusted source (NASA corpus, ADS paper, SDE document, or verified web source), it should be retained and evaluated. A single mention in a peer-reviewed paper indexed by ADS is sufficient evidence to retain a candidate.
- Abstain (return zero results) ONLY when no discovery channel across all steps yields any plausible candidate. Do not abstain if candidates were found — instead, include them with appropriate caveats about uncertainty.
- Dual-use or sensitive domains may be surfaced only with explicit caution.
- No implication of legality, safety, certification, or fitness for use.
