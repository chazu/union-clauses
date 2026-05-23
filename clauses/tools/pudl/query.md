# Querying via datalog rules

Rules live in `.pudl/schema/pudl/rules/*.cue` (repo-scoped, shadows global at `~/.pudl/schema/pudl/rules/`).

Rule shape (CUE):
```cue
#FooDependsOnBar: {
  head: { name: "depends", args: ["$x", "$y"] }
  body: [
    { name: "module", args: ["$x"] },
    { name: "import", args: ["$x", "$y"] },
  ]
}
```

Run: `pudl query <rule-name>` or `pudl facts query "..."`.

Shared `$Variables` between atoms = equi-join. Recursive rules supported (semi-naive fixpoint). Keep rules in version control if reproducibility matters.
