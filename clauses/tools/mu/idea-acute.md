# IDEA + ACUTE — the pudl/mu convergence loop

Two complementary models. **IDEA** describes the *kinds of knowledge* the system tracks. **ACUTE** describes the *pipeline phases* that act on that knowledge.

## IDEA — four layers of knowledge

| Layer | Holds | Pudl implementation |
|-------|-------|---------------------|
| **I**ntention | schemas, constraints, policies | `~/.pudl/schema/` (git-tracked CUE repo) |
| **D**efinition | desired-state values | `~/.pudl/schema/definitions/*.cue` (named instances) |
| **E**xecution | tool-computed results | mu manifests imported as `pudl/mu.#Manifest` |
| **A**pplication | actual live state | raw imported data (JSON/YAML/NDJSON), bitemporally versioned |

Mu owns **E** (writes manifests). Pudl owns **I**, **D**, **A** (and the analysis layer that joins them all).

## ACUTE — five-phase pipeline

| Phase | Action | Owner |
|-------|--------|-------|
| **A**ccumulate | import actual state from live systems | pudl (`pudl import`, `mu observe \| pudl import`) |
| **C**onfigure | normalize imported data, resolve naming | pudl (schema inference, definition matching) |
| **U**nify | compare desired vs actual, detect drift | pudl (CUE unification, drift detection) |
| **T**ransform | export drifted resources as convergence targets | pudl (`pudl export-actions --drifted`) |
| **E**xecute | run convergence actions, report results | mu (`mu build --emit-manifest`) |

Loop closes when manifest is re-ingested → re-observe → next iteration.

## The loop visualized

```
   ┌──────────────────────────────────────────────────────────┐
   ▼                                                          │
[pudl: Application layer]                                     │
  state imported from live systems                            │
       │                                                      │
       │ U: pudl unifies with Definition layer                │
       ▼                                                      │
[pudl: drift detected]                                        │
       │                                                      │
       │ T: pudl export-actions --drifted                     │
       ▼                                                      │
[mu.json — desired targets]                                   │
       │                                                      │
       │ E: mu build --emit-manifest                          │
       ▼                                                      │
[manifest.json — Execution layer]                             │
       │                                                      │
       │ A: pudl import manifest + mu observe                 │
       ▼                                                      │
[pudl: refreshed Application layer]                           │
       │                                                      │
       └──────────────────────────────────────────────────────┘
```

## What each tool can answer

**Pudl can answer** (across all four IDEA layers):
- What should exist (Intention + Definition)
- What does exist (Application)
- What changed and when (bitemporal queries)
- Where contracts are violated (BRICK validation)
- What needs to converge (Unify result)

**Mu can answer** (Execution layer only):
- What happened in the last build (manifest)
- What is cached vs needs to rebuild
- What plugins reported as current state (observe)

## When you need the full loop

- Live infrastructure that drifts (cloud resources, k8s, files)
- Multi-step convergence where each step depends on observed state of the prior
- Audit trail of every change to declared state

## When you don't

- Pure builds with no live state — just `mu build`, no pudl needed
- Read-only analysis of a repo — just `pudl import`, no mu needed

## Authoritative

```
mu guide pudl                # cycle walkthrough from mu's side
pudl guide mu                # cycle walkthrough from pudl's side
pudl guide drift             # drift detection mechanics
```

Recipes: see `personal:tools/mu/converge-recipes`.
