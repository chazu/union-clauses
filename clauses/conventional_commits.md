## Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/) for all commits.

Format: `<type>(<scope>): <description>`

Common types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `build`, `ci`.

- Keep the description in the imperative mood ("add", not "added") and under 72 characters.
- Use `!` after the type/scope or a `BREAKING CHANGE:` footer for breaking changes.
- Scope is optional but encouraged when the change is localized to a package, module, or subsystem.

Examples:
- `feat(parser): support trailing commas in tuple literals`
- `fix(api): handle 429 from upstream with exponential backoff`
- `refactor!: drop support for Node 18`
