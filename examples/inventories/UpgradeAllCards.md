# Legacy Package Inventory: UpgradeAllCards

Archive snapshot:

```text
UpgradeAllCards.dll
UpgradeAllCards.pck
```

Observed characteristics:

- pack version `2`
- Godot `4.5.1`
- legacy embedded manifest
- type-level `[ModInitializer]` in the assembly
- almost no Godot-side content beyond metadata

This remains evidence that code-heavy mods can use very small PCKs. The current loader still requires an external manifest, and the DLL must be rebuilt if its referenced STS2 signatures changed.
