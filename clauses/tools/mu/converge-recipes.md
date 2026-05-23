# Converge recipes — copy-paste ACUTE loop

## One-shot full converge (drift → apply → re-observe)

```bash
# T: export drifted resources as mu targets
pudl export-actions --drifted > /tmp/converge.json

# Safety: plan first, inspect what would happen
mu build --plan --config /tmp/converge.json //...

# E: execute convergence + emit manifest
mu build --emit-manifest --config /tmp/converge.json //... > /tmp/manifest.json

# A: ingest manifest back into pudl
pudl import /tmp/manifest.json --origin mu

# A: re-observe to confirm convergence
mu observe --json --config /tmp/converge.json //... | pudl import --origin mu
```

## Observe-only (drift check without applying)

```bash
mu observe --json --config /tmp/converge.json //... | pudl import --origin mu
pudl facts list --relation drift
```

## Partial converge (specific target)

```bash
pudl export-actions --drifted --target //k8s/myapp > /tmp/c.json
mu build --plan --config /tmp/c.json //k8s/myapp
mu build --emit-manifest --config /tmp/c.json //k8s/myapp > /tmp/m.json
pudl import /tmp/m.json --origin mu
```

## When loop stalls (target keeps drifting after apply)

```bash
pudl drift                                  # see what's still drifted
pudl facts list --relation drift            # full drift history
pudl facts show <drift-id>                  # specific drift record
mu observe --verbose --config /tmp/c.json //target  # what plugin actually sees
```

Common causes:
- Plugin reports state in a different shape than pudl expects (`_schema` mismatch)
- Definition unintentionally non-idempotent (timestamps in config)
- External actor mutating resource between observe and apply

## Plan diff between iterations

```bash
mu build --plan --json --config /tmp/c1.json //... > /tmp/plan1.json
# ... after a change ...
mu build --plan --json --config /tmp/c2.json //... > /tmp/plan2.json
diff <(jq -S . /tmp/plan1.json) <(jq -S . /tmp/plan2.json)
```

## CI-style: build + push cache + emit manifest

```bash
mu build --emit-manifest //... > manifest.json
mu cache push ghcr.io/org/mu-cache
pudl import manifest.json --origin mu --source ci
```

## Loop with observation between iterations (slow but safest)

```bash
while true; do
  pudl export-actions --drifted > /tmp/c.json
  count=$(jq '.targets | length' /tmp/c.json)
  [ "$count" = "0" ] && break
  mu build --emit-manifest --config /tmp/c.json //... > /tmp/m.json
  pudl import /tmp/m.json --origin mu
  mu observe --json --config /tmp/c.json //... | pudl import --origin mu
done
```

Stops when no drift remains.

## Safety habits

- **Always `--plan` before applying** on convergence targets (anything impure)
- **Pass `--source mu` or `--source ci`** on pudl imports so attribution is preserved
- **Inspect `mu cache size`** periodically; cache grows fast on convergence loops
- **Use `pudl observe` (the `kind=fact`) to record manual interventions** that bypass the loop, so future drift detection sees them
