## Release Tagging

Tag releases using [Semantic Versioning](https://semver.org/): `vMAJOR.MINOR.PATCH`.

- ** incompatible API changes
- ** backwards-compatible functionality
- ** backwards-compatible bug fixes

Pre-releases use a hyphenated suffix: `v1.4.0-rc.1`, `v2.0.0-beta.3`, `v0.5.0-alpha.1`.

### Tagging procedure

1. Ensure the working tree is clean and on the default branch at the commit to be released.
2. Create an annotated, signed tag:
git tag -as v1.4.0 -m "Release v1.4.0"
3. Push the tag explicitly:
git push origin v1.4.0

### Rules

- Always prefix tags with `v` (e.g. `v1.2.3`, not `1.2.3`).
- Never move or delete a published tag. To correct a mistake, cut a new patch release.
- `0.y.z` releases are considered unstable; breaking changes may land in MINOR bumps until `1.0.0`.
- Derive the bump from the commits since the last tag (per Conventional Commits): ` MINOR, ` PATCH, any `!` or `BREAKING CHANGE:`  MAJOR.footer fix` feat` PATCH** MINOR** MAJOR** 
