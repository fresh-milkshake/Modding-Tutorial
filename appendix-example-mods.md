# Appendix B. Release Package Patterns

This appendix summarizes release-payload patterns observed in working STS2 mods without treating any one project as canonical.

## Why This Matters

Observed release archives show:

- more than one top-level zip layout
- embedded manifests and, sometimes, loose manifests
- tiny manifest-only `.pck` files and much larger Godot-heavy `.pck` files
- code-only releases and content-bearing releases

Archive shape and installed shape are not the same concern.

## Common Archive Patterns

### Pattern A: root payload

```text
MyMod.dll
MyMod.pck
```

### Pattern B: named subfolder

```text
MyMod/
  MyMod.dll
  MyMod.pck
```

### Pattern C: installer-friendly archive

```text
mods/
  MyMod/
    MyMod.dll
    MyMod.pck
    mod_manifest.json
```

All three can exist as release archives. For the local `v0.99.1` build documented here, the installed layout should still be normalized to root-level files under `mods\`.

## Reliable Takeaways

1. STS2 tolerates more than one release-zip layout.
2. A tiny `.pck` can still be a valid mod payload.
3. A loose `mod_manifest.json` is optional for payload packaging.
4. Matching the game's current Godot line is still the safest default for new work.
5. A `.pck` can contain much more than icons and strings.

## Inventory Data

Repo-local unpacked inventories still exist under `examples/` for evidence review, but the tutorial itself should be built from the generalized patterns above, not from any single local project.
