## Commit Granularity

Keep commits small and focused. Each commit should represent **one significant, logically coherent change** to the codebase.

### Size

- Target **100 lines changed** per commit whenever possible (additions + deletions, excluding generated files, lockfiles, and vendored code).
- If a change naturally exceeds this, split it along logical seams rather than padding the limit. Hard caps are less important than  a 200-line commit that does one thing is better than two artificially halved commits.

### Scope

- One commit, one concern. Do not mix refactors with feature work, formatting changes with logic changes, or unrelated fixes in a single commit.
- If the commit message needs the word "and" to describe what it does, it should probably be two commits.
- Pure formatting, renames, and mechanical refactors land in their own commits so that reviewable changes aren't buried in noise.
- Dependency bumps, generated file updates, and config changes are separate from the code changes that motivate them, unless they are inseparable.

### Practical guidance

- Stage hunks deliberately (`git add -p`) rather than committing whole files when a working tree contains multiple concerns.
- When a task requires several steps, plan the commit sequence up front: each commit should leave the tree in a buildable, test-passing state.
- If you discover an unrelated issue mid-task, either stash it for a follow-up commit or note it for  do not fold it into the current one.later coherence 80
