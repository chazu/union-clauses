# Mu ↔ pudl integration

Three JSON documents flow between the tools. Mu is the **Execution** layer; pudl is the **Knowledge** layer (Intention + Definition + Application).

## The three docs

### 1. `mu.json` (pudl → mu) — desired state

```
pudl export-actions --drifted > /tmp/converge.json
```

Emits drifted resources as mu targets. Pudl maps CUE schema prefixes to mu toolchain names:

| CUE prefix | mu toolchain |
|------------|--------------|
| `file.*`, `config.*` | `file` |
| `k8s.*`, `kubernetes.*` | `k8s` |
| `terraform.*`, `tf.*` | `terraform` |
| `shell.*`, `exec.*` | `shell` |
| (unknown) | `generic` |

Consumed via `mu build --config <file> //...`.

### 2. Manifest (mu → pudl) — execution result

```
mu build --emit-manifest --config /tmp/converge.json //... > /tmp/manifest.json
```

JSON, schema `mu.build.manifest/v1`. Auto-matches `pudl/mu.#Manifest` on import:

```
pudl import /tmp/manifest.json --origin mu
```

BRICK metadata (kind, implements) round-trips.

### 3. Observe (mu → pudl, fast path) — current state

Optional alternative to a full converge cycle when you only need to refresh observations:

```
mu observe --json --config /tmp/converge.json //... | pudl import --origin mu
```

Each plugin's `observe` method returns `current.records[]` with `_schema` fields — pudl routes them to the right schema automatically.

## Design principle

Pudl emits **desired state**, not drift diffs. The file plugin receives `{"path": "...", "content": "..."}` — knows nothing about CUE, drift, or pudl. **Any mu plugin works whether the target came from pudl or a hand-written `mu.cue`.**

This decoupling is intentional: mu plugins stay simple, pudl stays general.

## Boundary of responsibility

| Concern | Owner |
|---------|-------|
| What should exist | pudl (Intention + Definition) |
| What does exist | pudl (Application, via mu observe) |
| Drift detection | pudl (Unify phase) |
| Convergence plan | pudl emits desired targets; plugin emits actions |
| Convergence execution | mu (Execute phase) |
| BRICK contract validation | pudl (CUE unification) |
| BRICK metadata round-trip | both (mu carries it through manifest) |

## Authoritative

```
mu guide pudl                # mu's view
pudl guide mu                # pudl's view
```

See also: `personal:tools/mu/idea-acute`, `personal:tools/mu/brick`, `personal:tools/mu/converge-recipes`.
