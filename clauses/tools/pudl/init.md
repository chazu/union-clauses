# Initializing pudl

Pudl has two scopes: **global workspace** (`~/.pudl/`, shared across all repos) and **repo-scoped overlay** (`.pudl/` at repo root, optional).

## First-time global init

Once per machine:

```
pudl init           # creates ~/.pudl/ with schema git repo + CUE module + sqlite
pudl doctor         # verify health
pudl prime          # print canonical agent guide (read this)
```

Re-init: `pudl init --force` (destructive, prefer not).

## Per-repo setup

From repo root:

```
pudl repo init        # creates .pudl/ marker + .claude/skills/ for agent integration
pudl repo validate    # validate all repo schemas + definitions
```

`pudl repo init` creates:
- `.pudl/` — repo overlay marker
- `.claude/skills/` — PUDL skill files for agents

Then decide gitignore policy:
- **Commit `.pudl/`** if team shares schemas/rules (reproducible queries)
- **Gitignore `.pudl/`** if pudl is your personal tool here

Start importing: `pudl import --path <file>` — entries land in the global catalog.
Verify: `pudl list` shows entries; `pudl schema list` shows assigned schemas.

Re-init: `pudl repo init --force`.

See `personal:tools/pudl/ingest` for directory layout and verify-ingest commands. If `pudl` is missing, see `personal:tools/pudl/troubleshoot`.
