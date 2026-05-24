# Authoring a mu plugin

## Minimal plugin in Babashka (~30 lines)

`plugins/hello/plugin.bb`:

```clojure
#!/usr/bin/env bb
(require '[cheshire.core :as json])

(defn handle [req]
  (case (get req "method")
    "discover" {"name"             "hello"
                "version"          "0.1.0"
                "protocol_version" 1
                "consumes"         []
                "produces"         ["text"]
                "capabilities"     ["discover" "plan"]
                "config_schema"    {"message" {"type" "string"}}}
    "plan"     (let [target (get req "target")
                     msg    (get-in target ["config" "message"] "hello world")]
                 {"actions" [{"id"         "echo"
                              "command"    ["echo" msg]
                              "inputs"     {}
                              "outputs"    []
                              "depends_on" []}]
                  "declared_outputs" {}})
    {"error" (str "unknown method: " (get req "method"))}))

(loop []
  (when-let [line (read-line)]
    (println (json/generate-string (handle (json/parse-string line))))
    (flush)
    (recur)))
```

## Plugin's own `mu.cue`

`plugins/hello/mu.cue`:

```cue
package mu

plugin: {
  entrypoint: "plugin.bb"
  toolchain:  "bb"                 // runtime (bb, sh, python3); empty = direct exec
  files:      ["plugin.bb"]        // bundled files; empty = all in dir
  guide:      "GUIDE.md"           // auto-included
  schemas: [                       // optional vendored CUE
    {module: "mu/hello", version: "v1", path: "schemas/mu/hello"}
  ]
}

// How to build the plugin bundle itself
targets: [{
  target:    "build"
  toolchain: "shell"
  sources:   ["plugin.bb"]
  config:    {command: ["true"], impure: false}
}]
```

## Register in project root `mu.cue`

```cue
plugins: [{ name: "hello", script: "plugins/hello/plugin.bb" }]

targets: [{
  target:    "//greet"
  toolchain: "hello"
  sources:   []
  config:    { message: "hi from mu" }
}]
```

## Test it

```
mu plugin test hello           # protocol smoke test
mu build --verbose //greet     # see NDJSON I/O
```

## Authoring checklist

1. **discover** returns name, version, protocol_version: 1, capabilities, config_schema
2. **plan** returns actions[] + declared_outputs
3. Plugin handles unknown methods with `{"error": ...}`
4. NDJSON: one JSON object per line, flush after each write
5. Read stdin until EOF, then exit cleanly
6. For convergence: set `impure: true` on side-effecting actions
7. For observation: implement `observe`, tag records with `_schema` for pudl routing
8. For secrets: implement `resolve_secret` + `store_secret` together; never log values
9. Write a `GUIDE.md` — surfaced via `mu guide plugin <name>`

## Languages

Any executable works. Bundled examples in `~/dev/go/mu/plugins/`:
- Babashka (`.bb`) — most common, fast startup
- Shell (`.sh`) — minimal, fine for thin wrappers
- Compiled binary — for hot-path plugins

`plugin.entrypoint` + `plugin.toolchain` together determine how mu invokes it. Empty `toolchain` = direct exec.

## Distribution

Plugin can be:
- Local file: `script: "plugins/hello/plugin.bb"`
- OCI bundle: `url: "oci://..."`, `digest: "sha256:..."` (built via mu, pushed via `mu cache push`)
- Inline command: `command: ["sh", "-c", "..."]` (toy plugins only)

## Pith VM plugins (inline)

Targets can carry `plan: "..."` / `transform: "..."` pith programs that replace plugin dispatch entirely. Useful for one-off logic where a full plugin is overkill. See `mu guide pith-plugins`.

