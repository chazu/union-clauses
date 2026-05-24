# Mu ↔ pudl integration

Mu = **Execution** layer. Pudl = **Knowledge** layer (Intention + Definition + Application). This clause covers: IDEA/ACUTE model, BRICK classification, and the 3 JSON docs that flow between tools.

## IDEA — four layers of knowledge

| Layer | Holds | Pudl implementation |
|-------|-------|---------------------|
| **I**ntention | schemas, constraints, policies | `~/.pudl/schema/` (git-tracked CUE repo) |
| **D**efinition | desired-state values | `~/.pudl/schema/definitions/*.cue` |
| **E**xecution | tool-computed results | mu manifests imported as `pudl/mu.#Manifest` |
| **A**pplication | actual live state | raw imported data, bitemporally versioned |

Mu owns **E**. Pudl owns **I**, **D**, **A**.

## ACUTE — five-phase pipeline

| Phase | Action | Owner |
|-------|--------|-------|
| **A**ccumulate | import actual state from live systems | pudl (`pudl import`, `mu observe \| pudl import`) |
| **C**onfigure | normalize imported data, resolve naming | pudl |
| **U**nify | compare desired vs actual, detect drift | pudl (CUE unification) |
| **T**ransform | export drifted resources as convergence targets | pudl (`pudl export-actions --drifted`) |
| **E**xecute | run convergence actions, report results | mu (`mu build --emit-manifest`) |

Loop closes when manifest re-ingested → re-observe → next iteration.

## The loop

```
   ┌──────────────────────────────────────────────────────────┐
   ▼                                                          │
[pudl: Application layer]                                     │
       │ U: pudl unifies with Definition layer                │
       ▼                                                      │
[pudl: drift detected]                                        │
       │ T: pudl export-actions --drifted                     │
       ▼                                                      │
[mu.json — desired targets]                                   │
       │ E: mu build --emit-manifest                          │
       ▼                                                      │
[manifest.json — Execution layer]                             │
       │ A: pudl import manifest + mu observe                 │
       ▼                                                      │
[pudl: refreshed Application layer] ──────────────────────────┘
```

## BRICK — composable infrastructure blocks

Classification system for infra/code blocks. Pudl validates BRICK constraints via CUE unification; mu executes resulting actions. **Mu does NOT enforce BRICK — pudl does.**

### Four kinds

| kind | what it is |
|------|-----------|
| `relationship` | typed link between two blocks (depends-on, exposes, owns) |
| `interface` | contract — fields + constraints other blocks must satisfy |
| `component` | concrete implementation claiming to satisfy an interface |
| `kit` | curated bundle of interfaces + components + relationships |

Components carry `implements: "//interface/name"`. Pudl validates via CUE unification at planning time.

### Five registers

| Register | pudl side | mu side |
|----------|-----------|---------|
| Building block | CUE definition | Target in mu.cue |
| Role | CUE schema type (interface, component, kit) | Toolchain name |
| Implementation | (delegates to mu) | Plugin |
| Configuration | CUE values, constraints | `config{}` map on target |
| Kit | CUE package / workspace | mu.cue file + plugins/ dir |

### When BRICK matters

Use BRICK kinds when:
- Multiple components could satisfy same contract (swap implementations without rewriting consumers)
- Want pudl to surface contract violations as drift

Skip when: one-off resources with no contract; pure build targets (Go binaries, Docker images) — leave `kind` unset.

## Three JSON docs

### 1. `mu.json` (pudl → mu) — desired state

```
pudl export-actions --drifted > /tmp/converge.json
```

Pudl maps CUE schema prefixes to mu toolchain names:

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
pudl import /tmp/manifest.json --origin mu
```

Schema `mu.build.manifest/v1`. Auto-matches `pudl/mu.#Manifest`. BRICK metadata (kind, implements) round-trips.

### 3. Observe (mu → pudl, fast path) — current state

```
mu observe --json --config /tmp/converge.json //... | pudl import --origin mu
```

Plugin's `observe` returns `current.records[]` with `_schema` fields — pudl routes them.

## Design principle

Pudl emits **desired state**, not drift diffs. File plugin receives `{"path": "...", "content": "..."}` — knows nothing about CUE, drift, or pudl. **Any mu plugin works whether the target came from pudl or hand-written `mu.cue`.**

## What each tool answers

**Pudl**: what should exist, what does exist, what changed and when, where contracts violated, what needs to converge.

**Mu**: what happened in last build (manifest), what is cached vs needs rebuild, what plugins reported as current state (observe).

## When you need the full loop

- Live infrastructure that drifts (cloud, k8s, files)
- Multi-step convergence depending on observed state
- Audit trail of every change

## When you don't

- Pure builds, no live state → just `mu build`
- Read-only analysis of repo → just `pudl import`

## Authoritative

```
mu guide pudl          # mu's view
pudl guide mu          # pudl's view
pudl guide drift       # drift mechanics
pudl guide schemas     # CUE side of BRICK
```

Recipes: see `personal:tools/mu/converge-recipes`.
