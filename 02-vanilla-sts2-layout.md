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

At a high level, this layout separates the game into three modding-relevant domains. `data_sts2_windows_x86_64/` is the managed runtime payload and therefore the authoritative source for assemblies, public types, and method signatures. `SlayTheSpire2.pck` is the primary Godot content pack and therefore the authoritative source for asset names, localization layout, and `res://` paths. `mods/` is the local installation target for user-provided mod payloads.

The remaining directories matter less for day-to-day mod authoring. `controller_config/` is part of the shipped game, but it is not central to the standard C# mod workflow described here. `tools/` may exist in a local install and may contain useful helpers, but those helpers should not be treated as a universal requirement unless you have confirmed they are part of the exact game build you are documenting.

## Release Metadata

`release_info.json` is the authoritative local version marker. On this machine it reports `v0.99.1` dated `2026-03-13T20:40:28-07:00`.

This file is more important than it first appears. A large share of STS2 modding advice is version-sensitive: loader behavior, hook signatures, content conventions, and even install layout expectations can drift between Early Access updates. If a tutorial makes a precise statement, that statement should be traceable to a concrete game version, and `release_info.json` is the simplest local anchor for that purpose.

## Managed Runtime Payload

The most important files under `data_sts2_windows_x86_64/` are:

- `sts2.dll`
- `0Harmony.dll`
- `GodotSharp.dll`

In practice, this directory is the reference surface for nearly every code-facing task. It is where you point ILSpy or another decompiler when you need to inspect the live API. It is where your IDE resolves symbols when you want compile-time correctness against the exact runtime the game is using. It is also where you obtain the game-shipped Harmony and GodotSharp assemblies, which is usually safer than mixing in copies from unrelated installs.

This distinction matters because STS2 modding is not built on a separate public SDK. The installed game is the SDK. Once that is clear, many workflow choices become simpler: compile against the live assemblies, inspect the live assemblies, and assume that any version mismatch between your project and the local install is a likely source of failure.

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

Taken together, these members describe a real loader rather than a loose community convention. `LoadModsInDirRecursive` shows that STS2 performs recursive discovery. `TryLoadModFromPck` shows that `.pck` files are not an accidental packaging trick but a first-class payload form understood by the game itself. `CallModInitializer` shows that initialization is structured around a known attribute-based contract rather than a guessed entrypoint name. `GetModdedLocTables` shows that the loader has explicit knowledge of modded localization rather than treating strings as a side effect of asset loading.

This is the main reason the tutorial stays close to the live API. When the game already models loading, initialization, and localization explicitly, the safest modding strategy is usually to align with those existing contracts instead of inventing a parallel system.

## Loader State And Consent

`ModSettings` and `ModManager` also expose:

- `PlayerAgreedToModLoading`
- disabled-mod tracking

That means local mod loading is not merely a matter of dropping files into a directory. The game maintains player-facing state around whether mod loading is allowed and which mods are disabled. This matters for troubleshooting because a correctly packaged mod can still fail to appear if the relevant consent or enablement path is not in the expected state. It also matters for documentation, because a tutorial that describes only file layout and omits loader state is incomplete.

## What Logs And The Main Pack Confirm

From runtime logs and `SlayTheSpire2.pck`:

- the game runs on a custom Mono-enabled Godot 4.5.1 line
- vanilla localization lives under `localization/<lang>/...`
- relic art uses Godot-imported resources such as `images/relics/*.png.import`

These findings are useful because they connect the code-facing and content-facing sides of modding. Runtime logs tell you what the executable actually launched, including the engine line and practical loading behavior. The main pack tells you how the shipped game names and stores its resources. Together they form a better source of truth than any single community guide.

When documentation and behavior disagree, trust the live install first. In STS2 modding, a decompiled method, a shipped pack path, and a runtime log line usually outweigh any remembered convention.
