# When to reach for mu (and when not)

## Use mu standalone when

- Hermetic multi-language builds with a shared cache
- Need sandbox guarantees (no env inheritance, no net by default)
- Distribute build artifacts via OCI registry
- Compose actions from heterogeneous tools into one DAG
- Want a single command runner that respects content-addressing semantics

## Use mu + pudl when

- Drift-converge against live infra (k8s, terraform, files, remote hosts)
- Need a bitemporal record of every desired/observed state transition
- BRICK-modeled systems where contract satisfaction must be validated
- Multiple feedback loops (observe → unify → converge → observe)
- Cross-repo / cross-system queries over what was built / observed / converged

## Use mu inline pith plans when

- One-off transform logic that doesn't justify a full plugin
- Quick prototyping before extracting a real plugin
- Inline `plan:` / `transform:` programs on a target

## Skip mu entirely when

- Pure single-language inner dev loop, native incremental tool is faster (`go test ./...`, `cargo check`)
- No need for hermetic exec, sandbox, or content-addressed caching
- Stateless ephemeral builds with no infra side effects

## Skip pudl when (but keep mu)

- No live infra to drift against
- Don't need the bitemporal store
- Build outputs are self-contained artifacts, not convergence side effects

## Skip mu when (but keep pudl)

- Just importing/querying data
- No need to execute anything against real systems
- Ad-hoc analysis of structured documents

## Anti-patterns

- **Wrapping every shell script as a mu target** — overhead with no payoff. Reserve for things that benefit from caching, sandbox, or convergence semantics.
- **Using mu for things that change every run** — without `impure: true`, you'll get stale cache hits. With `impure: true`, you lose caching. Ask if mu is the right tool.
- **Skipping `--plan` on convergence targets** — apply-without-plan against infra is how prod accidents happen.
- **Running converge loops without pudl** — without pudl recording state, you have no audit trail and no drift detection between runs.
