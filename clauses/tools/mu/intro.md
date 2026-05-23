# Mu — when to use it

Mu = language-agnostic, plugin-driven build coordinator. NOT `go build` / `cargo` / `make`. Knows nothing about languages, compilers, toolchains. External plugins emit action subgraphs over NDJSON; mu orchestrates them as one DAG of content-addressed actions and executes hermetically.

## Reach for mu when this repo needs

- Hermetic multi-language builds with a shared cache (Go + Zig + Docker + shell, one DAG)
- Drift-converge against live infrastructure (k8s, terraform, files) — pairs with pudl for the full ACUTE loop
- BRICK-composed systems (interface/component/kit modeling) — pudl validates, mu executes
- Sandbox guarantees (Linux namespaces / macOS Seatbelt / copy isolation)
- OCI-layout artifact distribution (push/pull builds across machines)

## Do NOT use mu for

- Single-language inner-dev-loop where the native tool is already fast (just use `go test ./...`)
- Stateless ephemeral builds with no infra side effects
- Workflows that need imperative control flow (mu plans static DAGs)

## Bootstrap pointers

```
mu --version
mu guide                 # topic index — authoritative
mu guide overview        # mental model
mu guide pudl            # how mu pairs with pudl
```

`mu guide <topic>` is the single source of truth. When these clauses disagree with `mu guide`, trust the guide.
