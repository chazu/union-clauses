# mu.cue config schema

Root config, auto-discovered by walking up from cwd. Subdirectory `mu.cue` files auto-merged.

## Top-level fields

```cue
package mu

toolchains:   [...]   # scratch-built tool downloads
plugins:      [...]   # external plugin registrations
targets:      [...]   # declared build units
cache:        {...}   # OCI cache config (backends, read_repair, write_through, push)
secrets:      {...}   # write-policy (writable_refs: ["pass:bootstrap/*", ...])
advice:       [...]   # lifecycle observer plugins (advise method)
preprocessor: {...}   # transform non-JSON config (extension, command)
```

## Toolchain entry

```cue
{
  toolchain: "name"
  from:      "scratch"
  config: {
    version:      "x.y.z"
    url:          "https://..."
    sha256:       "..."
    strip_prefix: "optional/leading/dir"
  }
}
```

mu downloads, verifies, extracts, registers files as artifacts. Available to plugins as `toolchain_artifacts` in plan requests.

## Plugin entry

One of:
```cue
{ name: "go", script:  "plugins/go/plugin.bb" }   # local file
{ name: "k8s", url:    "oci://...", digest: "sha256:..." }   # remote OCI
{ name: "shell", command: ["sh", "-c", "..."] }   # inline
```

## Target entry

```cue
{
  target:    "//path/name"        // unique within project
  toolchain: "plugin-name"
  sources:   ["glob/**", "file.go"]
  deps:      ["//other:target"]
  config:    { ... }              // plugin-specific, validated against config_schema
  sealed_inputs:       { NAME: "scheme:path" }
  sealed_input_modes:  { NAME: "env" | "file" }
  sealed_outputs:      { NAME: "scheme:path" }
  sealed_output_modes: { NAME: "create" | "overwrite" | "create_if_absent" }

  // BRICK metadata (used by pudl, ignored by mu execution)
  kind:       "relationship" | "interface" | "component" | "kit"
  implements: "//interface/name"

  // Optional inline pith VM programs (replace plugin dispatch)
  plan:      "..."   // pith program emits actions
  transform: "..."   // runs after deps complete
}
```

## Config validation

mu sends `discover` to plugins at startup, captures their `config_schema` (JSON-Schema Draft-7 subset), and validates each target's `config{}` block at planning time. Errors caught before any action runs.

## Naming conventions

- Target names: `//<path>/<name>` (Bazel-style)
- Wildcard: `//...` (all targets recursively), `//pkg:...` (all in package)
- Plugin names: lowercase, hyphens (`remote-exec`, not `RemoteExec`)
