## Changelog

If a `CHANGELOG.md` exists in the repository root, update it as part of wrapping up any task that changes user-facing behaviour, fixes a bug, modifies a public interface, refactors existing code, or updates documentation.

If no `CHANGELOG.md` exists, create one before adding your entry. Use the following as the initial structure:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com).

## [Unreleased]
```

Follow the [Keep a Changelog](https://keepachangelog.com) format. Add your entry under the `## [Unreleased]` section. Use the appropriate subsection: **Added**, **Changed**, **Fixed**, **Removed**, **Security**, or **Refactored**. Write each entry as a single line from the perspective of a user or operator — what changed and why it matters, not how it was implemented. If a Jira ticket is associated with the work, append the key in brackets, e.g. `[PLAT-1234]`.
