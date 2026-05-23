# Mu CLI surface

## Commands

```
mu build [targets...]       # plan + execute DAG
mu scratch                  # bootstrap toolchains (download/extract/verify)
mu observe [targets...]     # query plugins for current state (drift detection)
mu target list              # list all declared targets in mu.cue
mu graph [target]           # show dependency chain (ASCII / DOT / JSON)
mu cache <sub>              # ls | inspect | size | clean | push | login | logout
mu plugin <sub>             # list | info | status | test | push | add
mu verify                   # check CAS blob integrity + schema namespaces
mu guide [topic]            # built-in reference
mu version
```

## Key flags

| flag | applies to | what |
|------|------------|------|
| `--plan` | build | dry-run DAG, no exec |
| `--emit-manifest` | build | emit JSON manifest (pudl-consumable) |
| `--json` / `--ndjson` | most | machine-readable output |
| `--config <file>` | build, observe | use external mu.json (from pudl export-actions) |
| `--no-cache` | build | force rebuild, skip CAS lookups |
| `--no-discover-cache` | build | bypass cached plugin discover responses |
| `--jobs N` | build | parallel worker count (default = CPU) |
| `--verbose` | most | show plugin I/O |
| `--reverse` | graph | show what depends on target |
| `--dot` | graph | Graphviz output |
| `--fix` | verify | delete corrupt blobs |

## Guide topics worth knowing

`mu guide` — index. Topics: overview, mu.cue, plugins, build, observe, pudl, cache, secrets, secret-gen, toolchains, shell, protocol, secret-providers, pith-plugins, sandbox, advice, plugin &lt;name&gt;.

Always run `mu guide <topic>` before guessing.
