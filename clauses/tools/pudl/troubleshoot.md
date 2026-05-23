# Pudl gotchas

- `pudl: command not found` — install from `~/dev/go/pudl` via `go install ./cmd/pudl`
- Workspace seems broken → `pudl doctor` checks dirs, DB integrity, schema repo, git init, orphans
- Schema names look weird → normalized to `<package>.#<Definition>`; `pudl schema list` shows canonical form
- Empty query results → `pudl facts stats` to confirm facts exist; `pudl list` to confirm imports
- Rule changes not picked up → repo-scoped `.pudl/schema/pudl/rules/` shadows global with same filename; check both
- New rule file not loaded → `pudl rule add <file>` (or `--global`) registers it
- Migration errors on DB open → auto-migration retries safely. If persistent: back up `~/.pudl/data/sqlite/catalog.db`, delete, re-import
- Cannot use CGo SQLite drivers — pudl uses `modernc.org/sqlite` exclusively
- `pudl observe` records as `chazu` (or `$USER`) by default → agents must pass `--source <agent-name>` or attribution is lost
- `pudl status` ≠ workspace health. It reports drift convergence status for definitions. Use `pudl doctor` for health.
- Stale schema inference after editing CUE → `pudl schema reinfer` re-runs inference on existing entries
