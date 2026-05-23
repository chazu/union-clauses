# Mu plugins — overview

Plugins are external executables that translate `target` declarations into action subgraphs. mu speaks NDJSON to them over stdin/stdout.

## Bundled plugins (in `~/dev/go/mu/plugins/`)

**Build:**
- `go` — Go binaries (cross-compile, tags, ldflags, race)
- `zig` — Zig builds
- `docker` — Docker image builds
- `cowsay` — Demo transformer

**Convergence (impure side effects):**
- `file` — write/copy/symlink/delete local files
- `k8s` — Kubernetes resource convergence + drift detection
- `terraform` — infra provisioning + state capture
- `remote-exec` — SSH command execution
- `remote-file` — SSH file convergence
- `lint` — linter wrapper (observe + fix modes)

**Secret providers:**
- `pass` — password-store backend
- `sops` — SOPS-encrypted files
- `keypair-gen` — generate ed25519/ECDSA keypairs

**Infrastructure observers (observe only):**
- `host` — remote host over SSH (OS, packages, services)
- `aws` — AWS resources (EC2, VPC, subnets)

**Other:**
- `void` — webhook reporter (advice plugin)
- `scratch` — toolchain bootstrap (also built-in)
- `shell` — built-in, no binary (runs arbitrary commands)

## Discovering installed plugins

```
mu plugin list                # all registered
mu plugin info <name>         # discover response + metadata
mu plugin status <name>       # runtime state
mu guide plugin <name>        # plugin's GUIDE.md
mu plugin test <name>         # protocol smoke test
```

## Adding a plugin

```
mu plugin add <path-or-url>   # register
```

Or declare in `mu.cue` `plugins[]` (see `personal:tools/mu/cue-config`).

## Writing one

See `personal:tools/mu/plugin-protocol` (wire format) and `personal:tools/mu/plugin-authoring` (step-by-step).
