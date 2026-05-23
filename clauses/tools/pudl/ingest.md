# Ingesting this repo into pudl

Basic: `pudl pull <path-or-url>` — auto-detects format, infers schema, content-addresses each record.

Per-source workflow:
1. `pudl sources add <name> <path>` — register a source
2. `pudl pull <name>` — re-ingest (idempotent; only new content gets new IDs)
3. `pudl schema list` — confirm inferred schema
4. `pudl facts stats` — verify counts

Re-ingest is safe: content hash dedupes, bitemporal layer tracks change.

Repo-scoped config lives in `.pudl/` at repo root. Add to `.gitignore` unless team shares ingest config.
