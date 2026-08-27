# Idea Bundle MCP reference

## Canonical ZIP

Use exactly the repository Idea Bundle V2 contract:

```text
manifest.json
sources/*.md
entities/source-materials.json
entities/source-chunks.json
entities/ideas.json
entities/provenance.json
entities/relations.json
```

The manifest and every entity file use deterministic SHA-256 checksums. Stable IDs are opaque. Do not derive IDs from labels, slugs, filenames, or chapter titles.

## Quality gates

MCP checks schema, canonical checksum, source checksums, exact chunk offsets, course and chapter scope, published-ready status, source-grounded provenance, stable keys, relation endpoints, duplicate relations, and prerequisite cycles. The content gate also checks meaningful source chunk coverage, Idea provenance coverage, duplicate or near-duplicate canonical statements, and formula trace warnings.

`quality.valid` must be true before submission. A warning remains visible to the admin and must be reviewed, even if it is not blocking.

## MCP sequence

1. Receive a validated `sourcePackToken` from `source.ingest`.
2. Start `idea_product` with the opaque course and chapter choices (`workflow.start`).
3. Obtain scope and contract with `workflow.get_run` and `workflow.get_contract`.
4. (Optional) Inspect source chunks and existing coverage with `knowledge.list_sources`, `knowledge.get_source`, `knowledge.search_source_chunks`, and `analysis.get_coverage`.
5. Build the Idea Bundle from the locked Source Pack and run contract.
6. Call `authoring.validate_idea_bundle` repeatedly during revision.
7. Call `authoring.submit_idea_bundle` only after validation is green.
8. Wait for admin review and publication before Product authoring.
