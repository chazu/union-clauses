# Logging observations

Observations are structured assertions about code, recorded as facts in the bitemporal store. They are the raw material for downstream datalog rules, the nous reasoning engine, and cross-repo analysis.

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

## Scope format

`<repo>:<path>` — `repo` is the repository name, `path` is package/file within it. Omit path for repo-wide observations.

```
pudl:internal/database      # specific package
pudl:cmd/api                # command package
pudl                        # whole repo
myapp:pkg/auth              # different repo
                            # (omit scope entirely for cross-repo/global)
```

Agents must use this format so observations join cleanly across repos via datalog.

## Source

Always pass `--source <your-agent-name>` (e.g. `--source claude-code`). Default is the OS user. Without explicit source, observations look human-authored and lose attribution for filtering / weighting.

## When to log an observation

Log when you notice something worth remembering past the current turn:

- You discovered a non-obvious invariant or constraint while reading code
- You hit an obstacle that future-you (or another agent) will hit too
- You spotted a pattern that repeats and deserves a name
- You spotted an antipattern worth flagging before someone copies it
- You found a bug that won't be fixed in this session
- You have a concrete suggestion that doesn't fit in the current task

Do NOT log:
- Routine task progress (use the task tool / commit messages)
- Speculation without evidence
- Anything trivially re-derivable by reading code (`func X exists in file Y`)
- Conversational asides
- The same observation already logged — query first: `pudl facts list --relation observation --source <you>`

## Scope: same repo vs. across repos

**Same repo (most common)**
The observation is about the repo you're working in. Use `<thisrepo>:<path>`.
```
pudl observe "Config struct has 47 fields — split by concern" \
  --kind suggestion --scope myapp:internal/config --source claude-code
```

**About pudl itself** (or any tool you depend on)
You're in `myapp` but noticed something about pudl, mu, or another tool. Scope it to that tool's repo so it lands where pudl-maintainers (or the next agent in pudl) will see it.
```
pudl observe "pudl import silently skips files >100MB" \
  --kind bug --scope pudl:internal/importer --source claude-code
```
The observation lives in the global catalog regardless of which repo's cwd you ran it from. Scope = where it applies, not where you observed it.

**Cross-cutting / global**
Applies to multiple repos or your dev environment generally. Omit `--scope` or use a meta-scope.
```
pudl observe "all Go repos should use golangci-lint v2" \
  --kind suggestion --source claude-code
```

## Verifying + maintaining

```
pudl facts list --relation observation --source claude-code
pudl facts show <id>                       # full detail
pudl facts retract <id>                    # was wrong
pudl facts invalidate <id>                 # was true, no longer is
```

Retraction preserves history (bitemporal): the fact still exists at its old transaction time, just no longer asserted now. This is the right move when later evidence contradicts an earlier observation — don't delete, retract.

## Why bother

Single observations are cheap and feel low-value. Compounding value comes from:
- Datalog rules that derive higher-level findings from many observations
- Time-travel queries: "what did we know about auth on 2026-03-01"
- Cross-repo joins: "which repos have the same antipattern"
- Continuity: next agent picks up where you left off without re-reading everything

Log the observation. Future-you will thank present-you.
