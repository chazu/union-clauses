# Common pudl recipes

## Snapshot data and diff later

```
pudl import --path snapshot.json
# ... time passes, re-import same logical thing ...
pudl import --path snapshot.json
pudl facts list --relation observation --as-of-valid 2026-01-01
```

Bitemporal store retains old versions automatically — no manual diffing.

## Record an observation

See `personal:tools/pudl/observations`.

## Join two sources via datalog

Import both. Write a rule whose body has two atoms with a shared `$var` on the join key. Install with `pudl rule add` and run with `pudl query <relation>`.

## Find orphans / dangling refs (recursive rule)

```
reachable($x) :- root($x).
reachable($y) :- reachable($x), refs($x, $y).
orphan($x)    :- node($x), not reachable($x).
```

Then `pudl query orphan`.

## Inspect catalog

See `personal:tools/pudl/ingest` ("Verify ingest succeeded").

## Reset

- Repo overlay: `rm -rf <repo>/.pudl/` — drops local schemas, rules, marker. Catalog untouched.
- Global catalog: `rm -rf ~/.pudl/data/` — drops all imported data and facts. Schemas survive.
- Nuclear: `rm -rf ~/.pudl/ && pudl init` — fresh workspace.
