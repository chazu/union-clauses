# Pudl — when to use it

Pudl = local content-addressed catalog + bitemporal fact store + datalog query engine. Pure-Go SQLite, no daemon, no server. Lives at `~/.pudl/`.

Reach for pudl when this repo needs:
- Stable IDs for unstructured/semi-structured artifacts (docs, configs, code chunks) that survive renames
- "What did this look like on date X" — bitemporal history without git archaeology
- Cross-source joins (e.g. issues × PRs × deploy logs) via datalog rules
- Schema-validated ingestion of CSV/JSON/YAML/NDJSON without writing parsers
- Recording observations about code (patterns, bugs, antipatterns) in a queryable store

Do NOT use pudl for:
- Primary app storage (use your real DB)
- Anything needing concurrent writes from multiple processes
- Streaming/realtime (batch ingest model)

## Verify install + bootstrap

```
pudl --version
pudl prime          # canonical agent-facing prompt — read this first
pudl guide          # topic-based reference (overview, import, schemas, facts, datalog, ...)
pudl doctor         # workspace health check
```

`pudl prime` is the single source of truth for the CLI surface. When in doubt, run it. When these clauses disagree with `pudl prime`, trust prime.
