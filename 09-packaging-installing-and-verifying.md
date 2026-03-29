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

This recommendation is about reliability, not theoretical possibility. The game may tolerate more than one payload arrangement, and release archives in the wild certainly vary, but installation is where ambiguity becomes user-visible. A normalized root-level install shape gives the loader and installed-mod UI the clearest possible input.

## Release Archive Recommendation

Archive shape can vary, but the cleanest release zip for local users is install-ready root files:

```text
MyMod.json
MyMod.dll
MyMod.pck
README.txt
```

That lets the user copy the payload directly into `<STS2>\mods\`.

The less a user has to infer about final placement, the fewer support problems the release archive creates.

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

This checklist is designed to catch the most common class of STS2 packaging failure: a release that is individually plausible at every layer but inconsistent across layers. The `.dll` may be current while the `.pck` is stale, or the payload names may differ only slightly, or the installer metadata may point to the wrong shape.

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

The important idea here is replacement, not accumulation. Leaving behind stale files from an older payload can create a mixed install that neither the old nor the new release actually intended.

## Removal Workflow

To remove a local mod:

1. close STS2
2. delete the mod's root-level files from `mods\`
3. remove external settings only if you want a full reset
4. relaunch and verify vanilla behavior

This matters because a mod can have both installed payload files and external state. Removing only one of those layers may be correct for an upgrade but not for a true reset.

## Verification Checklist

After install, verify:

1. the game launches cleanly
2. the mod loader recognizes the mod
3. the initializer clearly ran
4. gameplay behavior changed as expected
5. strings render correctly
6. images render correctly
7. no new errors appear in the logs

This is a runtime checklist, not just a packaging checklist. The fact that files landed in the expected directory does not by itself prove that the mod is correctly installed.

## Practical Shipping Rule

Do not ship from memory. Before every release:

1. inspect the staged output manually
2. inspect the `.pck` contents
3. install the exact release payload into a clean local `mods\` root
4. launch STS2 and verify the exact build you are about to distribute

This final step is often the difference between a package that looks correct and a package that is actually correct.
