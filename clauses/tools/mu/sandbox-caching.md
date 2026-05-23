# Mu sandbox + caching

## Sandbox

Actions execute with isolation:

- **Linux**: namespaces (mount, pid, net)
- **macOS**: Seatbelt profile
- **Fallback**: copy isolation (less strict, used when above unavailable)

Hermetic guarantees:
- `env` is the entire environment — no inheritance from caller
- Only declared `inputs` are visible
- `network: false` (default) blocks net; set `network: true` only when needed (and only with `retries > 0` allowed)

## Caching

Content-addressed store at `~/.mu/cache/` (OCI layout). Action cache key = hash of (command + env + input digests + sealed_input refs + sealed_output refs + modes).

- **Cache hit**: action skipped, outputs restored from CAS
- **Cache miss**: action runs, outputs hashed + stored
- **Impure** (`impure: true`): never cached, always runs

## Cache management

```
mu cache ls                  # list blobs
mu cache inspect <digest>    # blob metadata
mu cache size                # total bytes
mu cache clean               # prune unused (interactive)
mu cache push <ref>          # push to remote OCI registry
mu cache login <registry>    # auth
mu cache logout <registry>
```

## Remote cache (OCI registry)

```cue
cache: {
  backends: [{
    type: "oci"
    ref:  "ghcr.io/org/mu-cache"
  }]
  read_repair:   true   // pull missing blobs from remote on cache miss
  write_through: true   // push every new blob immediately
  push:          true   // enable push side
}
```

Lets a CI build hydrate developer machines (and vice versa).

## Verify integrity

```
mu verify                    # check all CAS blobs
mu verify --fix              # delete corrupt blobs
```

Run after disk errors or aborted operations.

## When to bypass cache

- `--no-cache` — force rebuild this run (debugging plugin output)
- `--no-discover-cache` — bypass cached plugin `discover` (after editing plugin)
- Set `impure: true` on the action — never cache (convergence side effects)

## Authoritative

```
mu guide cache               # cache config reference
mu guide sandbox             # isolation modes
```
