# Mu secrets — sealed_inputs & sealed_outputs

Sealed I/O routes secret values through provider plugins. Values **never** enter the cache, manifest, or logs. Refs and modes ARE part of the cache key (so they affect re-execution decisions).

## Reading secrets — `sealed_inputs`

Per-target or per-action:

```cue
sealed_inputs: {
  KUBECONFIG_TOKEN: "pass:deploy/k8s-token"
  AWS_SECRET_KEY:   "pass:aws/secret-key"
}
sealed_input_modes: {
  KUBECONFIG_TOKEN: "env"   // exported as $KUBECONFIG_TOKEN
  AWS_SECRET_KEY:   "file"  // path written to $AWS_SECRET_KEY
}
```

mu calls provider's `resolve_secret` before action runs, injects per the mode.

## Writing secrets — `sealed_outputs`

```cue
sealed_outputs: {
  ADMIN_PASS: "pass:registry/admin"
}
sealed_output_modes: {
  ADMIN_PASS: "create_if_absent"   // also: create, overwrite
}
config: {
  command: ["sh", "-c", "openssl rand -base64 24 > $MU_SEALED_OUT_DIR/ADMIN_PASS"]
}
```

Action writes to `$MU_SEALED_OUT_DIR/<NAME>`; mu routes through provider's `store_secret`. `sealed_outputs` forces `impure: true` (never cached — store side effect must fire).

## Ref scheme grammar

`<provider>:<path>` — e.g. `pass:foo/bar`, `sops:secrets.yaml#key`. Provider-specific. Some support variants (`pass:` first-line vs `pass:raw:` full content).

## Project write policy

```cue
secrets: {
  writable_refs: [
    "pass:bootstrap/*",
    "sops:secrets/generated.yaml#*"
  ]
}
```

mu rejects `sealed_outputs` whose refs don't match a glob. Prevents accidental writes outside designated paths.

## secret-gen toolchain (built-in)

Mints secrets from derivation commands without a plugin binary:

```cue
{
  target:    "//secrets/api-key"
  toolchain: "secret-gen"
  sealed_outputs: { OUT: "pass:api/key" }
  sealed_output_modes: { OUT: "create_if_absent" }
  config: {
    derivation: ["openssl", "rand", "-hex", "32"]
  }
}
```

Idempotent with `create_if_absent` — won't regenerate if already present.

## Providers (bundled)

- `pass` — password-store
- `sops` — Mozilla SOPS encrypted files
- `keypair-gen` — ed25519/ECDSA keypairs

## Authoring a new provider

Implement `resolve_secret` + `store_secret` together. Declare both in `capabilities`. See `personal:tools/mu/plugin-protocol` and `mu guide secret-providers`.

## Authoritative

```
mu guide secrets             # user reference
mu guide secret-gen          # built-in derivation toolchain
mu guide secret-providers    # implementing providers
```
