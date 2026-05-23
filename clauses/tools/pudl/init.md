# Initializing pudl

Pudl has two scopes: **global workspace** (`~/.pudl/`, shared across all repos) and **repo-scoped overlay** (`.pudl/` at repo root, optional).

## First-time global init

Once per machine:

```
pudl init           # creates ~/.pudl/ with schema git repo + CUE module + sqlite
pudl doctor         # verify health
pudl prime          # print canonical agent guide (read this)
```

What `pudl init` creates:
- `~/.pudl/schema/` — git-backed CUE module for schemas
- `~/.pudl/data/sqlite/catalog.db` — the catalog
- `~/.pudl/data/raw/YYYY/MM/DD/` — raw imported payloads
- `~/.pudl/config.yaml` — global config

Re-init: `pudl init --force` (destructive, prefer not).

## Per-repo setup

No `pudl init` flag for repo scope. To opt a repo into pudl-aware workflows:

1. `mkdir -p .pudl/schema/pudl/rules` — repo-scoped rules dir (shadows global)
2. Decide gitignore policy:
   - **Commit `.pudl/`** if team shares schemas/rules (reproducible queries)
   - **Gitignore `.pudl/`** if pudl is your personal tool here
3. Start importing: `pudl import --path <file>` — entries land in the global catalog, tagged by source
4. Verify: `pudl list` shows imported entries; `pudl schema list` shows assigned schemas

## Verify install before doing anything

```
pudl --version
pudl doctor         # workspace health (dirs, DB, schema repo, orphans)
pudl guide          # topic-based reference index
pudl prime          # canonical agent-facing prompt — single source of truth
```

If `pudl` missing: build from `~/dev/go/pudl` via `go install ./cmd/pudl` (Go module name is `pudl`, not a github path).

## Key directories at a glance

```
~/.pudl/                       # global workspace
  schema/                      # CUE module, git-versioned
  schema/pudl/rules/           # global datalog rules
  data/sqlite/catalog.db       # the catalog
  data/raw/                    # immutable imported payloads
  config.yaml

<repo>/.pudl/                  # optional repo overlay
  schema/pudl/rules/           # repo rules (shadow global)
  schema/<pkg>/*.cue           # repo-pinned schemas
```

There is no separate repo catalog — facts live in the global DB. Isolate by source tags / definition scopes, not by separate DBs.
