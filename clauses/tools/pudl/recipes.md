# Common pudl recipes

## Snapshot repo state for diffing later
```
pudl pull . --source repo-snapshot
# ... time passes ...
pudl facts query 'changed_since("2026-01-01")'
```

## Join two sources
Register both → write rule with shared variable on the join key → `pudl query`.

## Find orphans / dangling refs
Recursive rule: `reachable($x) :- root($x). reachable($y) :- reachable($x), refs($x, $y).` Then `orphan($x) :- node($x), not reachable($x).`

## Reset everything
`rm -rf .pudl/` (repo-scoped only). Global cache: `rm -rf ~/.pudl/data/`.
