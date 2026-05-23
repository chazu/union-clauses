# Defining schemas for this repo

Pudl infers schemas on pull. To pin them:
1. `pudl schema export <name> > .pudl/schema/<pkg>/<name>.cue`
2. Edit CUE definition (`<package>.#<Definition>` form — names normalized automatically)
3. `pudl schema validate` before re-pulling

Optional metadata fields: `SchemaType`, `ResourceType`, `BaseSchema`, `IdentityFields`, `TrackedFields`, `IsListType`.

`IdentityFields` controls dedup. `TrackedFields` controls what triggers a new fact version on re-ingest. Get these wrong = noisy fact churn.
