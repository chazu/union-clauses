# Querying pudl

Two query surfaces: direct fact listing (`pudl facts`) and derived datalog rules (`pudl query`).

## Direct fact queries

```
pudl facts list --relation observation              # all observations
pudl facts list --relation observation --source claude-code
pudl facts list --relation depends                  # any relation
pudl facts show <id>                                # full fact detail
pudl facts stats                                    # aggregate counts
pudl facts retract <id>                             # mark as wrong
pudl facts invalidate <id>                          # mark as no longer true
```

Temporal flags on `facts list` / `query`:
- `--as-of-valid <time>` — what was true at that time
- `--as-of-tx <time>` — what we believed at that time
- both — what we believed at tx-time about valid-time

## Datalog rules

Rules live in `.pudl/schema/pudl/rules/*.cue` (repo-scoped, highest priority) or `~/.pudl/schema/pudl/rules/` (global). Repo rules shadow global with same name.

Install a rule:
```
pudl rule add <file.cue>             # repo-scoped
pudl rule add <file.cue> --global    # global
```

Rule shape (CUE):
```cue
#FooDependsOnBar: {
  head: { name: "depends", args: ["$x", "$y"] }
  body: [
    { name: "module", args: ["$x"] },
    { name: "import", args: ["$x", "$y"] },
  ]
}
```

Run:
```
pudl query depends                          # all results of relation
pudl query depends --x=foo                  # filter by positional field
pudl query depends -f /tmp/adhoc.cue        # load rule file ad-hoc
```

Shared `$Variables` between atoms = equi-join. Recursive rules supported (semi-naive fixpoint). Commit rules to version control if reproducibility matters.

Add `--json` to anything for machine-readable output.
