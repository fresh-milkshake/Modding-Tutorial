# Appendix B. Example Mods

All archives from `EXAMPLE_MODS` were unpacked into:

```text
Modding Tutorial/examples/unpacked/
```

Per-mod inventories live in:

```text
Modding Tutorial/examples/inventories/
```

## Why These Examples Matter

These mods are useful because they show what working STS2 releases actually look like in the wild:

- different archive top-level layouts
- different `.pck` contents
- embedded vs loose manifests
- code-only vs asset-bearing mods
- localization-bearing vs non-localized mods

## High-Level Comparison

| Mod | Archive layout | Loose manifest | Embedded manifest | Notable observations |
| --- | --- | --- | --- | --- |
| BetterSovereignBlade | subfolder | no | yes | includes image import and minimal code stubs in `.pck` |
| BetterSpire2 | root payload | no | yes | `.pck` built with Godot 4.3.0 |
| DamageMeter | `mods/DamageMeter/` | yes | yes | richest example; ships localization, source files, settings docs |
| typing | root payload | no | yes | manifest contains inline comments, likely editor-exported |
| UndoAndRedo | root payload | no | yes | very small `.pck`, manifest-only |
| UpgradeAllCards | root payload | no | yes | smallest clean starter example |

## Strongest Lessons From The Example Set

### 1. STS2 tolerates multiple release zip layouts

You should not assume that every mod archive must look the same. The loader-facing payload may still work even if the archive packaging style differs.

### 2. `.pck` size does not tell you whether a mod is "real"

`UpgradeAllCards.pck` and `UndoAndRedo.pck` are tiny and mostly manifest-driven.

`DamageMeter.pck` is large and contains substantial Godot-side content.

Both patterns are valid.

### 3. A loose `mod_manifest.json` is optional in practice

Only `DamageMeter` ships one loose next to the payload. The others rely on an embedded manifest inside the `.pck`.

### 4. Matching the game's Godot version is still the safest default

One example pack was built with Godot 4.3.0 and still exists as a working release, but the current game pack is Godot 4.5.1. For new work, match the current game line unless you have a tested reason not to.

### 5. Real mods may ship more than assets inside `.pck`

`DamageMeter` embeds many source `.cs` files, localization JSON, and project metadata. That is a useful reminder that STS2 mod `.pck` payloads are general Godot content packs, not just icon containers.

## Recommended Examples To Study First

If you are new to STS2 modding, study the examples in this order:

1. `UpgradeAllCards`
   - smallest clean payload
   - easiest to reason about
2. `BetterSpire2`
   - simple code-focused mod with embedded manifest
3. `DamageMeter`
   - full-featured example with localization, UI, settings, and installer-friendly packaging

## Inventory Links

- [BetterSovereignBlade.md](examples/inventories/BetterSovereignBlade.md)
- [BetterSpire2.md](examples/inventories/BetterSpire2.md)
- [DamageMeter.md](examples/inventories/DamageMeter.md)
- [typing.md](examples/inventories/typing.md)
- [UndoAndRedo.md](examples/inventories/UndoAndRedo.md)
- [UpgradeAllCards.md](examples/inventories/UpgradeAllCards.md)
