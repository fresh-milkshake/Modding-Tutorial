# Legacy Package Inventory: BetterSovereignBlade

Archive snapshot:

```text
BetterSovereignBlade/
  BetterSovereignBlade.dll
  BetterSovereignBlade.pck
```

Observed PCK characteristics:

- pack version `3`
- Godot `4.5.1`
- legacy embedded manifest
- icon resources and `project.binary`

This inventory predates the current external-manifest loader contract. To test the payload on `v0.107.1`, place it in a named mod directory and add a current `<ModId>.json` beside matching DLL/PCK basenames. This layout change alone does not establish binary compatibility.
