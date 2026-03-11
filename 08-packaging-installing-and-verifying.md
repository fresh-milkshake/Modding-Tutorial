# 08. Packaging, Installing, And Verifying

## Minimal Release Payload

For most STS2 mods, the minimum practical payload is:

```text
MyMod.dll
MyMod.pck
```

Recommended user-friendly payload:

```text
mods/
  MyMod/
    MyMod.dll
    MyMod.pck
    mod_manifest.json
```

That second layout is the easiest to explain to end users and matches the real `DamageMeter` release shape.

## Release Checklist

Before zipping a mod:

1. confirm the build targets the current STS2 assemblies
2. confirm the `.dll` is the version you just compiled
3. confirm the `.pck` contains the current manifest/assets/localization
4. confirm `pck_name` is aligned with your payload naming
5. confirm the initializer attribute and method name still match
6. confirm modded localization files are under `res://<pck_name>/localization/<lang>/...`
7. confirm you are not shipping stale or renamed files from an older build
8. confirm any updated relic art includes refreshed Godot import artifacts, not just a new PNG

## Local Installation

The default local install target is:

```text
<STS2 Game Root>\mods\
```

Common shapes:

### Folder copy

```text
<STS2 Game Root>\mods\MyMod\MyMod.dll
<STS2 Game Root>\mods\MyMod\MyMod.pck
```

### Root payload copy

Some release zips extract loose `.dll + .pck` at the archive root. In that case, place them inside a mod folder under `mods\` yourself unless the release documentation says otherwise.

## Upgrade Workflow

When updating a mod:

1. close STS2
2. replace the old mod payload with the new one
3. keep any mod-specific external settings files unless the update requires a reset
4. relaunch and verify the new behavior

Real example: `DamageMeter` keeps its settings under:

```text
%APPDATA%\Godot\app_userdata\DamageMeter\DamageMeter_settings.json
```

That pattern is mod-specific, not a universal STS2 rule.

## Removal Workflow

To remove a local mod safely:

1. close STS2
2. delete the mod's folder from `mods\`
3. remove any external settings/state only if you want a clean reset
4. relaunch and verify vanilla behavior

## Verification Checklist

After installation, verify all of these:

1. the game launches cleanly
2. the mod loader recognizes the mod
3. your initializer behavior clearly happened
4. gameplay changes are present
5. strings render correctly
6. images/icons render correctly
7. no new errors appear in `sts2_stdout.log` or `sts2_stderr.log`

## Common Failure Patterns

### Mod does not appear at all

Usually one of:

- wrong install path
- missing or malformed manifest
- bad `pck_name`
- broken initializer attribute or method name
- hard version mismatch

### Code appears to load, but strings are missing

Usually one of:

- stale `.pck`
- wrong localization file layout
- correct JSON, but packed outside `res://<pck_name>/localization/<lang>/...`
- wrong language code
- wrong keys

### Strings load, but images do not

Usually one of:

- wrong asset path
- wrong snake_case filename
- missing import metadata or stale Godot export

### Mod appears, but behavior does not change

Usually one of:

- initializer ran but patch registration failed
- patch target signature changed
- hook chosen was wrong for the actual gameplay event

### Hover or inspect UI breaks for a custom relic

Usually one of:

- the relic is not associated with any `RelicPool`, so `DynamicDescription` throws while resolving energy color formatting
- the small icon path works, but `BigIconPath` still points at a missing `images/relics/<IconBaseName>.png`

## Practical Advice For Shipping

Do not ship from memory. Before every release:

1. inspect the output folder manually
2. inspect the `.pck` contents
3. install the release payload into a clean local `mods\` folder
4. launch STS2 and verify the exact build you are about to distribute

That last step catches a surprising number of packaging mistakes.
