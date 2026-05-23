# Pudl gotchas

- `pudl: command not found` — install from `~/dev/go/pudl` via `go install ./cmd/pudl`
- Schema names look weird → normalized to `<package>.#<Definition>`; use `pudl schema list` to see canonical form
- Empty query results → check `pudl facts stats` first; verify ingest succeeded
- Rule changes not picked up → repo-scoped rules in `.pudl/schema/pudl/rules/` shadow global; check both
- Migration errors on open → DB is auto-migrating; safe to retry. If persistent: backup `~/.pudl/data/sqlite/catalog.db`, delete, re-pull
- Cannot use CGo SQLite drivers — pudl uses `modernc.org/sqlite` exclusively
