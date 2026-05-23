# Ingesting data into pudl

Ingest = `pudl import`. Auto-detects format (JSON, YAML, CSV, NDJSON), infers schema, content-addresses each record, writes raw payload to `~/.pudl/data/raw/YYYY/MM/DD/`.

## Basic usage

```
pudl import --path data.json              # single file, format auto-detected
pudl import --path "logs/*.json"          # wildcard batch import
pudl import --path data.json --schema aws.#Instance  # force a schema
cat data.json | pudl import --path -      # stdin (use '-')
```

## Verify ingest succeeded

```
pudl list                                 # show recent entries
pudl list --schema <name>                 # filter by schema
pudl show <id>                            # full entry + content
pudl facts stats                          # fact-store aggregate counts
```

## Envelope wire format

A JSON file shaped like:
```json
{"schema": {"module": "mu/aws", "version": "v1"},
 "definitions": [...],
 "data": <payload>}
```
is auto-detected. The CUE ref is recorded; inline `definitions` are cached for future imports. Plain JSON without envelope is imported untouched.

## Re-ingest semantics

Idempotent at the content level: identical bytes = same SHA256 = same ID = no new entry. Changes produce new fact versions in the bitemporal store.

## Where data lives

- Raw payloads: `~/.pudl/data/raw/YYYY/MM/DD/YYYYMMDD_HHMMSS_origin.ext`
- Metadata: `~/.pudl/data/metadata/`
- Catalog: `~/.pudl/data/sqlite/catalog.db` (single global DB; no per-repo catalog)

Repo-scoped config / schemas / rules live in `<repo>/.pudl/` after `pudl repo init` — facts still land in the global catalog.
