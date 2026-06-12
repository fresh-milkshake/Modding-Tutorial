# Legacy Package Inventory: DamageMeter

Archive snapshot:

```text
mods/
  DamageMeter/
    DamageMeter.dll
    DamageMeter.pck
    mod_manifest.json
```

Observed PCK characteristics:

- pack version `3`
- Godot `4.5.1`
- legacy loose and embedded manifests
- custom localization files, source files, imports, and `project.binary`
- settings stored outside the game root through Godot user data

The directory-first archive anticipated the current installation style, but its legacy manifest schema is not the `v0.103.3` schema. Replace it with `DamageMeter.json` containing current fields before compatibility testing.
