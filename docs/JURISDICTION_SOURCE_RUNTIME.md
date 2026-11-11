# Jurisdiction Source Runtime

WF-020 now resolves curated official sources before relying on discovered sources.

## Runtime order

1. Normalize state/county.
2. Look up the provider-neutral jurisdiction registry.
3. Add curated state sources and county sources.
4. Merge discovered official sources only when they are not duplicates.
5. Preserve source-resolution metadata for auditability.
6. Flag sources whose terms prohibit commercial live querying rather than silently using them.

The registry remains an accelerator, not an exhaustive national database. Unknown states/counties continue through controlled official-source discovery.

## Current curated registry

The acquisition repo currently contains curated entries for North Carolina state sources and Catawba, Caldwell, Mecklenburg, Wake, and New Hanover counties. The runtime contract is intentionally state/county agnostic so additional jurisdictions can be added without changing WF-020's search API.
