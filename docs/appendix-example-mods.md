# Appendix B. Release Package Patterns

Older community archives show several layouts, but the current loader contract is determined by `v0.103.3`, not by historical package contents.

## Current Install-Ready Pattern

```text
mods/
  MyMod/
    MyMod.json
    MyMod.dll
    MyMod.pck
```

The external JSON is required for discovery. The DLL and PCK are required only when `has_dll` or `has_pck` is true.

## Other Archive Shapes

Historical downloads may contain:

- root-level DLL and PCK files
- a named payload folder
- a prebuilt `mods/<Name>/` tree
- an embedded legacy `mod_manifest.json`
- a very small metadata-only PCK
- a large PCK containing art, localization, scenes, and source artifacts

These shapes describe how an archive was published. They do not prove that it satisfies the current loader.

## Migration Rule For Legacy Archives

To test an older DLL/PCK pair on the current build:

1. create `mods/<ModId>/`
2. add a current `<ModId>.json` manifest
3. rename payloads to `<ModId>.dll` and `<ModId>.pck`
4. set `has_dll` and `has_pck` accurately
5. rebuild against the current assemblies if the DLL does not load
6. inspect old PCK paths before assuming localization or art remains compatible

An embedded legacy manifest can remain as inert pack content, but it is not a substitute for the external current manifest.

## General Lessons

- archive layout and installed layout are separate concerns
- PCK size does not indicate whether a mod is valid
- a code-heavy mod may still need a PCK for localization or icons
- an asset-heavy mod may use little managed code
- exact internal paths matter more than external archive names
- old working releases are evidence for patterns, not current API authority

The files under `examples/` are preserved as historical package inventories. Read their manifest observations as snapshots of older releases.
