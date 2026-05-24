# Mu plugin protocol (NDJSON wire format)

Plugins read JSON requests from stdin (one per line) and write JSON responses to stdout (one per line). mu starts the plugin, sends `discover`, then sends `plan`/`observe`/etc. as needed. Plugin exits when stdin closes.

Errors at any point: respond with `{"error": "message"}`.

## Request envelope (unified)

```json
{
  "method": "discover|plan|observe|resolve_secret|store_secret|advise",
  "target": {...},
  "deps": [...],
  "toolchain_artifacts": {...},
  "secrets": {...},
  "secret_ref": "...",
  "secret_value": "...",
  "secret_mode": "create|overwrite|create_if_absent",
  "phase": "after-build",
  "manifest": {...},
  "advise_context": {...},
  "advise_config": {...}
}
```

## Six methods

### `discover` (required, called once at startup)

**Request:** `{"method": "discover"}`

**Response:**
```json
{
  "name": "plugin_name",
  "version": "0.1.0",
  "protocol_version": 1,
  "description": "one-line summary",
  "consumes": ["artifact:type"],
  "produces": ["artifact:type"],
  "capabilities": ["discover", "plan", "observe", "resolve_secret", "store_secret", "advise"],
  "config_schema": { ... JSON-Schema Draft-7 subset ... },
  "advise_phases": ["after-build"],
  "output_schema": {"module": "mu/plugin", "version": "v1", "definition": "#Type"}
}
```

Plugin must: declare every supported method in `capabilities`. Declare `config_schema` so mu can validate target.config before planning.

### `plan` (required, called once per target)

**Request:**
```json
{
  "method": "plan",
  "target": {
    "name": "//app",
    "toolchain": "go",
    "sources": [...],
    "config": {...},
    "sealed_inputs": {"NAME": "scheme:path"},
    "sealed_input_modes": {"NAME": "env" | "file"},
    "sealed_outputs": {"NAME": "scheme:path"}
  },
  "deps": [{"target": "//lib", "artifacts": {"binary": "lib.a"}}],
  "toolchain_artifacts": {"go": "/path/to/go"}
}
```

**Response:**
```json
{
  "actions": [{
    "id": "compile",
    "command": ["go", "build", "-o", "app"],
    "inputs": {"main.go": "main.go"},
    "outputs": ["app"],
    "depends_on": [],
    "env": {"GOOS": "linux"},
    "sealed_inputs": {"SSH_KEY": "pass:hosts/id"},
    "sealed_input_modes": {"SSH_KEY": "file"},
    "sealed_outputs": {"TOKEN": "pass:bootstrap/token"},
    "sealed_output_modes": {"TOKEN": "create_if_absent"},
    "network": false,
    "work_dir": ".",
    "impure": false,
    "timeout_s": 300,
    "retries": 3,
    "retry_backoff_ms": 100
  }],
  "declared_outputs": {"binary": "app"}
}
```

Plugin must: translate target → actions. Forward `sealed_inputs/outputs` from target onto actions that need them. Consume deps via the `artifacts` map. Populate `declared_outputs` (artifact-type → file path) so downstream targets can consume.

### `observe` (optional — drift detection)

**Request:**
```json
{
  "method": "observe",
  "target": {...},
  "secrets": {"AWS_SECRET_KEY": "<resolved>"},
  "toolchain_artifacts": {...}
}
```

**Response:**
```json
{
  "current": {
    "records": [
      {"_schema": "aws.ec2.instance", "instance_id": "i-abc", "state": "running"}
    ]
  }
}
```

Plugin must: query real system. Tag each record with `_schema` so pudl routes it correctly. Declare `"observe"` in capabilities.

### `resolve_secret` (optional — secret provider read side)

**Request:** `{"method": "resolve_secret", "secret_ref": "deploy/token"}`

**Response:** `{"value": "secret-bytes"}` or `{"error": "not found"}`

Plugin must: never log the value. Honor any ref grammar variants (e.g. `pass:` first-line vs `pass:raw:` full content). Document grammar in GUIDE.md.

### `store_secret` (optional — secret provider write side)

**Request:**
```json
{
  "method": "store_secret",
  "secret_ref": "deploy/token",
  "secret_value": "bytes",
  "secret_mode": "create" | "overwrite" | "create_if_absent"
}
```

**Response:** `{}` or `{"error": "ref exists"}`

Plugin must: implement all three modes. `create_if_absent` no-ops silently if ref exists.

### `advise` (optional — lifecycle observer)

**Request:**
```json
{
  "method": "advise",
  "phase": "after-build",
  "manifest": {full manifest},
  "advise_context": {
    "project_root": "...",
    "targets": ["//app"],
    "duration_s": 12.3,
    "git_sha": "abc123",
    "git_branch": "main",
    "git_dirty": false
  },
  "advise_config": {...},
  "secrets": {resolved sealed_inputs}
}
```

**Response:** `{"ok": true}` or `{"error": "..."}`

Plugin must: declare `advise_phases` in discover. Errors are non-fatal (logged, build proceeds). 30s timeout.

## Action shape — full field reference

| field | type | notes |
|-------|------|-------|
| `id` | string | unique within subgraph |
| `command` | []string | argv |
| `inputs` | map | name → path or `{action:other-id}` ref |
| `outputs` | []string | declared output file paths |
| `depends_on` | []string | intra-subgraph action IDs |
| `env` | map | full environment (no inheritance) |
| `sealed_inputs/outputs` | map | secret refs (never cached) |
| `sealed_input_modes` | map | `env` or `file` |
| `sealed_output_modes` | map | `create` / `overwrite` / `create_if_absent` |
| `network` | bool | grants net access in sandbox |
| `work_dir` | string | relative to project root |
| `impure` | bool | skip cache; forced true if sealed_outputs present |
| `timeout_s` | int | per-attempt; 0 = none |
| `retries` | int | extra attempts; only when network: true |
| `retry_backoff_ms` | int | between attempts |

## Design contracts

1. **Determinism** — plan response = pure function of request
2. **No post-construction DAG mutation** — plugins are planners + sensors, not orchestrators
3. **Secret hygiene** — values never in cache/manifest/logs; refs+modes are cache-key metadata
4. **Cross-target wiring** — producer's `declared_outputs` ↔ consumer's `deps[].artifacts`
5. **`sealed_outputs` forces `impure: true`** — store_secret side effect must always fire
