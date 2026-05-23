# Mu typical workflow

## From zero to first build

### 1. Write `mu.cue` at repo root

```cue
package mu

toolchains: [{
  toolchain: "bb"
  from:      "scratch"
  config: {
    version: "1.12.216"
    url:     "https://github.com/babashka/babashka/releases/download/v1.12.216/babashka-1.12.216-macos-aarch64.tar.gz"
    sha256:  "91499b3f430038f9b40e433215256a6e5392942780dca9984d493d2bcca7055d"
  }
}]

plugins: [{ name: "go", script: "plugins/go/plugin.bb" }]

targets: [{
  target:    "//cmd/hello"
  toolchain: "go"
  sources:   ["go.mod", "go.sum", "cmd/hello/main.go"]
  config:    { output: "hello", pkg: "./cmd/hello" }
}]
```

### 2. Bootstrap toolchains

```
mu scratch
```

Fetches URLs, verifies SHA-256, extracts, registers as artifacts. Once per machine + toolchain version.

### 3. Plan first (always, on convergence targets)

```
mu build --plan //cmd/hello
```

Shows DAG without executing. Use as safety check before any impure action.

### 4. Build

```
mu build //cmd/hello
```

Plugin starts → mu sends `discover` then `plan` for each target → DAG executes in parallel → outputs cached by input hash.

### 5. Inspect outputs

```
mu cache ls                       # list cached blobs
mu cache inspect <digest>         # blob metadata
mu graph //cmd/hello              # DAG view
mu graph //cmd/hello --reverse    # what depends on this
```

### 6. Emit manifest (when feeding pudl)

```
mu build --emit-manifest //cmd/hello > /tmp/manifest.json
```

JSON manifest with action outcomes, output digests, BRICK metadata. See `personal:tools/mu/pudl-integration`.

## Iterative tips

- Use `--no-cache` to force rebuild during plugin development
- Use `--verbose` to see plugin NDJSON I/O
- Use `--no-discover-cache` after editing a plugin's `discover` response
- Sub-`mu.cue` files in subdirectories auto-merge — split large configs
