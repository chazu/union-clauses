# When to add a schema (and when not to)

Pudl infers schemas on first pull. Inferred = fine for exploration. Pin when you need stability, validation, or shared meaning across sources.

## Add a schema when

- **Two sources should join** — pinned `IdentityFields` make the join key explicit; inferred keys drift between pulls
- **A field's meaning matters** — `status: "open"|"closed"|"merged"` should reject `"opn"` at ingest, not at query
- **You'll write rules against it** — rules reference field names; inferred names can shift if input shape changes
- **Records evolve over time** — `TrackedFields` controls which changes create new fact versions; without it, every cosmetic re-pull churns the fact store
- **Multiple repos ingest the same shape** — shared schema = shared queries
- **The data has identity beyond its hash** — e.g. "user #42" is the same user even if email changes; needs `IdentityFields: ["id"]`

## Skip the schema when

- One-shot exploration (`pudl pull`, poke around, forget)
- Source format is already authoritative elsewhere (e.g. typed JSON dump from another system)
- Data is genuinely free-form (notes, logs) — schema would lie
- You're still deciding what fields matter — premature pinning locks in wrong identity

## How to add one

1. Pull once unschematized: `pudl pull <source>`
2. Inspect inferred shape: `pudl schema export <name>`
3. Save to repo: `pudl schema export <name> > .pudl/schema/<pkg>/<name>.cue`
4. Edit the CUE:
   - Tighten types (`string` → enum, `int` → bounded range)
   - Set `IdentityFields` to the natural key (NOT content hash — hash is automatic). Controls dedup.
   - Set `TrackedFields` to fields whose changes matter (omit `updated_at`, `last_seen`, etc.). Controls fact churn.
   - Set `BaseSchema` if extending an existing definition (native CUE unification, no priority cascade)
   - Other optional metadata fields: `SchemaType`, `ResourceType`, `IsListType`
5. Validate: `pudl schema validate`
6. Re-pull: `pudl pull <source>` — now enforced

## Naming

Use `<package>.#<Definition>` form. Package = logical grouping (`github`, `slack`, `repo`), Definition = singular noun (`#Issue`, `#Message`, `#File`). Pudl normalizes via `schemaname.Normalize()` — pick the canonical form upfront to avoid alias confusion.

## Anti-patterns

- **Schematizing then never querying** — overhead with no payoff; leave inferred
- **`IdentityFields` = every field** — defeats dedup; pick the natural key
- **`TrackedFields` includes timestamps** — every re-pull = new fact version = exploding fact table
- **One mega-schema per source** — split by record type; CUE unification composes them
- **Editing inferred schema in place** — pin via export first, otherwise next pull may regenerate it
