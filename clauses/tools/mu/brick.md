# BRICK — composable infrastructure blocks

BRICK is a classification system for infra/code blocks. Pudl validates BRICK constraints via CUE unification; mu executes the resulting actions. **Mu does NOT enforce BRICK — pudl does.**

## Four kinds

| kind | what it is |
|------|-----------|
| `relationship` | a typed link between two blocks (depends-on, exposes, owns) |
| `interface` | a contract — fields + constraints other blocks must satisfy |
| `component` | a concrete implementation that claims to satisfy an interface |
| `kit` | a curated bundle of interfaces + components + relationships (a package) |

Components carry `implements: "//interface/name"` to declare contract satisfaction. Pudl validates this via CUE unification at planning time.

## Five registers (block / role / impl / config / kit)

| Register | pudl side | mu side |
|----------|-----------|---------|
| Building block | CUE definition | Target in mu.json |
| Role | CUE schema type (interface, component, kit) | Toolchain name (k8s, terraform, file, shell) |
| Implementation | (delegates to mu) | Plugin (plugins/k8s/plugin.bb, etc.) |
| Configuration | CUE values, constraints | `config{}` map on each target |
| Kit | CUE package / workspace | mu.json file + plugins/ dir |

## How BRICK metadata flows

```
pudl CUE definitions          # interfaces, components, kits declared
  ↓ pudl export-actions
mu.json targets               # each target carries kind + implements
  ↓ mu build --emit-manifest
manifest                       # kind + implements round-trip
  ↓ pudl import
pudl facts                    # contract satisfaction recorded as facts
```

Datalog rules over those facts can answer: "which components implement interface X", "which kits ship component Y", "is contract Z satisfied across all components".

## When BRICK matters

Use BRICK kinds when:
- Multiple components could satisfy the same contract (e.g. several k8s storage backends → one `storage` interface)
- You want to swap implementations without rewriting consumers
- You want pudl to surface contract violations as drift

Skip BRICK kinds when:
- One-off resources with no contract to satisfy
- Pure build targets (Go binaries, Docker images) — leave `kind` unset

## Authoritative

```
mu guide overview            # BRICK ecosystem context
pudl guide schemas           # CUE side of BRICK
```

See also: `personal:tools/mu/idea-acute` for how BRICK fits the IDEA/ACUTE model.
