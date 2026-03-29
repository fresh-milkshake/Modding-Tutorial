# 09. Packaging, Installing, And Verifying

## Installed Payload For The Current Local Build

For the local `v0.99.1` build documented here, the safest installed shape is:

```text
<STS2>\mods\
  MyMod.json
  MyMod.dll
  MyMod.pck
```

For code-only mods:

```text
<STS2>\mods\
  MyMod.json
  MyMod.dll
```

Keep `mod_manifest.json` inside the `.pck` or in staging. Do not copy loose `mod_manifest.json` files into the root `mods\` directory.

## Release Archive Recommendation

Archive shape can vary, but the cleanest release zip for local users is install-ready root files:

```text
MyMod.json
MyMod.dll
MyMod.pck
README.txt
```

That lets the user copy the payload directly into `<STS2>\mods\`.

## Release Checklist

Before shipping:

1. confirm the build targets the current STS2 assemblies
2. confirm the `.dll` is the one you just compiled
3. confirm the `.pck` contains the current manifest, assets, and localization
4. confirm `pck_name`, `<ModId>.json`, and payload basenames are aligned
5. confirm `<ModId>.json` correctly declares `has_dll` and `has_pck`
6. confirm the initializer name still matches the real method
7. confirm modded localization is packed under `res://<pck_name>/localization/<lang>/...`
8. confirm updated art includes refreshed import artifacts

## Local Install And Update

Default local target:

```text
<STS2 Game Root>\mods\
```

Update workflow:

1. close STS2
2. replace the old payload with the new root-level files
3. keep external settings only if the mod still expects them
4. relaunch and verify behavior

## Removal Workflow

To remove a local mod:

1. close STS2
2. delete the mod's root-level files from `mods\`
3. remove external settings only if you want a full reset
4. relaunch and verify vanilla behavior

## Verification Checklist

After install, verify:

1. the game launches cleanly
2. the mod loader recognizes the mod
3. the initializer clearly ran
4. gameplay behavior changed as expected
5. strings render correctly
6. images render correctly
7. no new errors appear in the logs

## Practical Shipping Rule

Do not ship from memory. Before every release:

1. inspect the staged output manually
2. inspect the `.pck` contents
3. install the exact release payload into a clean local `mods\` root
4. launch STS2 and verify the exact build you are about to distribute
