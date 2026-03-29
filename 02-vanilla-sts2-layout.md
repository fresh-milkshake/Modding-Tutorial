# 02. Game Layout And Loader Surface

## Verified Game Root Layout

The local install used here contains:

```text
Slay the Spire 2/
  controller_config/
  data_sts2_windows_x86_64/
  mods/
  tools/
  release_info.json
  SlayTheSpire2.exe
  SlayTheSpire2.pck
  sts2_stdout.log
  sts2_stderr.log
```

Practical meaning:

- `data_sts2_windows_x86_64/` is the managed runtime payload
- `SlayTheSpire2.pck` is the main Godot content pack
- `mods/` is the local install target for mods
- `tools/` can exist locally, but helper tooling there should not be treated as a vanilla requirement

## Release Metadata

`release_info.json` is the authoritative local version marker. On this machine it reports `v0.99.1` dated `2026-03-13T20:40:28-07:00`.

Use it whenever the tutorial makes a version-sensitive claim.

## Managed Runtime Payload

The most important files under `data_sts2_windows_x86_64/` are:

- `sts2.dll`
- `0Harmony.dll`
- `GodotSharp.dll`

This is the reference directory for decompilation, IDE symbol browsing, and mod compilation.

## Verified Modding Surface

The current build exposes these key types:

- `MegaCrit.Sts2.Core.Modding.ModManager`
- `MegaCrit.Sts2.Core.Modding.ModManifest`
- `MegaCrit.Sts2.Core.Modding.ModInitializerAttribute`
- `MegaCrit.Sts2.Core.Modding.ModHelper`
- `MegaCrit.Sts2.Core.Hooks.Hook`

Important `ModManager` members:

- `Initialize`
- `LoadModsInDirRecursive`
- `TryLoadModFromPck`
- `TryLoadModFromSteam`
- `CallModInitializer`
- `GetModdedLocTables`

Practical meaning:

- STS2 has a built-in managed mod loader
- `.pck` files are first-class mod payloads
- initialization is attribute-driven
- the loader understands both local mods and Steam Workshop mods

## Loader State And Consent

`ModSettings` and `ModManager` also expose:

- `PlayerAgreedToModLoading`
- disabled-mod tracking

So local mod loading is not just file discovery. There is user-facing state around consent and mod enablement.

## What Logs And The Main Pack Confirm

From runtime logs and `SlayTheSpire2.pck`:

- the game runs on a custom Mono-enabled Godot 4.5.1 line
- vanilla localization lives under `localization/<lang>/...`
- relic art uses Godot-imported resources such as `images/relics/*.png.import`

When documentation and behavior disagree, trust the live install first.
