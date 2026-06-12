# 09. Packaging, Installing, And Verifying

## Canonical Local Layout

For `v0.103.3`, use one directory per mod:

```text
<STS2>/mods/
  FieldNotes/
    FieldNotes.json
    FieldNotes.dll
    FieldNotes.pck
```

The loader recursively discovers the manifest and resolves both payloads in the same directory. A code-only mod omits the PCK and sets `has_pck` to `false`.

Root-level payloads remain technically valid, but directories prevent unrelated files from sharing one namespace and make removal atomic.

## Release Archive

An install-ready archive should preserve the `mods` boundary:

```text
mods/
  FieldNotes/
    FieldNotes.json
    FieldNotes.dll
    FieldNotes.pck
README.txt
```

Users can merge `mods/` into the game root without deciding where individual files belong.

## Build Validation

Before creating the archive:

1. delete and recreate `build/mods/<ModId>/`
2. build against the current `sts2.dll`, `0Harmony.dll`, and `GodotSharp.dll`
3. copy the resulting DLL as `<ModId>.dll`
4. create `<ModId>.pck` when `has_pck` is true
5. write `<ModId>.json`
6. fail the build if any declared payload is absent
7. list the PCK contents and verify required internal paths

The manifest should not claim optional files that are not present. The loader logs the mismatch and leaves the mod in an error state.

## Manifest Checklist

- `id` is stable and matches DLL/PCK basenames
- `version` matches the release being shipped
- `has_dll` and `has_pck` match actual files
- dependencies use exact manifest IDs
- `affects_gameplay` is `false` only for a genuinely non-gameplay utility
- description is short enough for the loader UI
- no unrelated `.json` files are inside the scanned install directory

## PCK Checklist

- localization is under `<id>/localization/<language>/`
- table filenames match vanilla names
- raw assets use unique namespaced paths
- fixed native lookup resources exist where the model expects them
- updated images include current Godot imports or bridge resources
- no development-only source, cache, or extracted vanilla files are included

## Clean Installation Test

Test the exact archive, not the source tree:

1. close STS2
2. move existing development copies of the mod outside `mods/`
3. extract the release archive into a clean game root
4. launch and confirm loader state
5. create a new run and exercise every custom content path
6. save, exit, relaunch, and continue the run
7. inspect all current logs for new errors

This test catches stale local files, unstaged resources, and accidental dependencies on the development directory.

## Updating

Replace the complete mod directory while the game is closed. Do not merge a new payload over an old directory when filenames or resource layout changed.

If a release renames model classes, manifest IDs, or localization slugs, document save compatibility explicitly. These identifiers can be persisted in profiles and runs.

## Removing

Close the game and remove `<STS2>/mods/<ModId>/`. External settings and saves are separate state and should be removed only when a full reset is intended.

Do not continue a run that owns custom models after removing the providing mod unless the game or the mod supplies a deliberate migration path.

## Release Evidence

Keep a short verification record containing:

- STS2 version and commit
- mod version
- staged DLL hash
- staged PCK hash
- tested install path
- tested new-run and save-load outcomes

The record turns a release from an assumed file set into a reproducible compatibility result.
