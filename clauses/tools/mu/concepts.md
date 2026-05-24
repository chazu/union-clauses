# Mu core concepts

## Five primitives

1. **Artifact** — content-addressed blob (SHA-256). Immutable. Stored in OCI layout at `~/.mu/cache/`. Distributable via OCI registry push/pull.
2. **Action** — hermetic transformation: input artifacts → output artifacts. Fields: `id`, `command`, `inputs` (name→digest), `outputs` (paths), `depends_on`, `env`, `network`, `impure`, `timeout_s`, `retries`, `sealed_inputs/outputs`. Cached by input hash if pure.
3. **Plugin** — external executable speaking NDJSON. Emits action subgraphs in response to `plan` requests.
4. **Target** — declared build unit in `mu.cue`. Has `target` (name like `//cmd/app`), `toolchain`, `sources`, `deps`, `config`, optional BRICK metadata (`kind`, `implements`), optional inline `plan`/`transform` pith programs.
5. **Toolchain** — special target with `from: "scratch"`. mu downloads, verifies SHA-256, extracts, registers files as content-addressed artifacts. Passed to downstream plugins via `toolchain_artifacts`.

## DAG model

Adjacency-list graph (`dag.Graph`). Topologically sorted. Workers execute in parallel up to `--jobs`. Each action's outputs become inputs to downstream actions; mu threads digests through the graph automatically.

## Pure vs impure

- **Pure** (default): same inputs → same outputs. Cached by input hash. Use for compilation, codegen.
- **Impure** (`impure: true`): external side effects. Never cached, always runs. Use for convergence (k8s apply, terraform apply, file writes). Forced when action has `sealed_outputs`.

## Manifest

JSON build result, schema `mu.build.manifest/v1`. Fields: `version`, `type`, `timestamp`, `duration_s`, `targets[]`, `actions[]`, `summary` (completed/cached/failed/cancelled). Each action: `id`, `cached`, `exit_code`, `outputs{name→digest}`. BRICK metadata round-trips. Consumed by `pudl import`.

## Where things live

```
~/.mu/cache/        # OCI layout, content-addressed
~/.mu/plugins/      # extracted plugins (after CAS bundle build)
<repo>/mu.cue       # root config (auto-discovered by walking up)
<repo>/**/mu.cue    # sub-configs (auto-merged)
```

Sandbox model: see `personal:tools/mu/sandbox-caching`.
