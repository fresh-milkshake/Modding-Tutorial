# 02. Game Layout And Loader Surface

## Installation Domains

A current Windows installation has three modding-relevant domains:

```text
Slay the Spire 2/
  data_sts2_windows_x86_64/
  mods/
  tools/
  release_info.json
  SlayTheSpire2.exe
  SlayTheSpire2.pck
  sts2_stdout.log
  sts2_stderr.log
```

`data_sts2_windows_x86_64/` is the managed runtime surface. `SlayTheSpire2.pck` is the native content surface. `mods/` is the discovery root for local mods. A failure should first be assigned to one of these domains before individual code is changed.

## Current Release Marker

The verified local `release_info.json` contains:

```json
{
  "commit": "460a0ece",
  "version": "v0.103.3",
  "date": "2026-05-29T13:36:05-07:00",
  "branch": "v0.103.3",
  "main_assembly_hash": -1584622192
}
```

This handbook was revalidated after Major Update #1. Public update notes identify `v0.103.2` as the main-branch consolidation of the preceding beta changes; the installed build used here is the later `v0.103.3` release.

## Loader Sequence

The current `ModManager` performs the following high-level sequence:

1. recursively enumerate files below `<STS2>/mods`
2. deserialize every `.json` file as a `ModManifest`
3. reject manifests without an `id`
4. order detected mods by their declared dependencies
5. load sibling `<id>.dll` and `<id>.pck` files according to `has_dll` and `has_pck`
6. call every type marked with `[ModInitializer]`
7. if no initializer exists, create a Harmony instance and call `PatchAll(assembly)` automatically
8. expose loaded mod assemblies to reflection and model discovery

The directory containing a manifest becomes that mod's payload directory. This is why a named subdirectory is now the clearest local installation unit.

## Public Modding Surface

The main types are:

- `MegaCrit.Sts2.Core.Modding.ModManager`
- `MegaCrit.Sts2.Core.Modding.ModManifest`
- `MegaCrit.Sts2.Core.Modding.ModInitializerAttribute`
- `MegaCrit.Sts2.Core.Modding.ModHelper`
- `MegaCrit.Sts2.Core.Hooks.Hook`
- `MegaCrit.Sts2.Core.Models.ModelDb`

Current public `ModManager` functionality includes loaded-mod enumeration, mod state, gameplay-relevant mod names, localization table discovery, and mod detection events. Several methods named in older guides, including direct public load and initializer helpers, are now implementation details and should not be treated as stable mod APIs.

## Dependencies

`dependencies` is a list of manifest IDs, not display names or filenames. The loader delays a mod until all declared dependencies are loaded. Missing and circular dependencies are reported as load failures.

Use dependencies only for real runtime requirements. A mod that merely interoperates with another mod should usually detect it at runtime instead of making it mandatory.

## Player Consent And Disablement

The loader also tracks player consent and disabled mods. A correct payload can remain inactive when mod loading has not been accepted or the mod is disabled in settings. Diagnose loader state before changing the DLL.

## Resource Packs

When `has_pck` is true, the loader calls `ProjectSettings.LoadResourcePack` for `<id>.pck`. The pack can add resources anywhere below `res://`, subject to Godot path and collision rules. Localization is more specific: the current loader probes `res://<id>/localization/<language>/<file>` for every loaded mod.

The next chapter turns this loader behavior into a project and installation layout.
