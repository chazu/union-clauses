# Logging observations

Observations are structured assertions about code, recorded as facts in the bitemporal store. Raw material for datalog rules and cross-repo analysis.

## Command

```
pudl observe "<description>" --kind <kind> --scope <repo>:<path> --source <agent>
```

## Kinds (pick the most specific)

| kind        | use for                                                |
|-------------|--------------------------------------------------------|
| fact        | a verified truth about the system                      |
| pattern     | a recurring structure or behavior worth naming         |
| antipattern | a recurring problem worth avoiding                     |
| obstacle    | something currently blocking progress                  |
| bug         | a defect (consider filing in issue tracker too)        |
| suggestion  | a proposed improvement                                 |
| opportunity | a potential enhancement, not yet a concrete suggestion |

Default kind is `fact`. Be deliberate — `fact` is the strongest claim.

## Scope

`<repo>:<path>` — repo name + package/file. Omit path for repo-wide; omit `--scope` entirely for cross-cutting/global.

Scope = where it applies, not where you observed it. Observations live in the global catalog regardless of cwd, so scope to the repo the observation is *about*: noticed a pudl bug while working in `myapp`? Use `--scope pudl:internal/importer`.

```
pudl observe "Config struct has 47 fields — split by concern" \
  --kind suggestion --scope myapp:internal/config --source claude-code
```

## Source

Always pass `--source <agent-name>` (e.g. `--source claude-code`). Default is `$USER` → looks human-authored, attribution lost.

## When to log

- Non-obvious invariant or constraint discovered while reading code
- Obstacle future-you (or another agent) will hit too
- Recurring pattern or antipattern worth naming
- Bug not fixed in this session
- Concrete suggestion that doesn't fit current task

Do NOT log:
- Routine task progress (use task tool / commits)
- Speculation without evidence
- Trivially re-derivable facts (`func X exists in file Y`)
- Duplicates — query first: `pudl facts list --relation observation --source <you>`

## Verify + maintain

```
pudl facts list --relation observation --source claude-code
pudl facts show <id>
pudl facts retract <id>      # was wrong
pudl facts invalidate <id>   # was true, no longer is
```

Retraction preserves bitemporal history — fact still exists at its old transaction time. Don't delete, retract.
