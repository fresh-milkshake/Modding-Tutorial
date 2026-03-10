# 02. Vanilla STS2 Layout

## Why This Matters

Many modding guides blur together three separate things:

- the shipped game
- community conventions
- local helper tooling

For STS2, you should keep them separate. The game itself is the authoritative source for runtime versions, assembly names, and asset/layout conventions.

## Verified Game Root Layout

The local install used for this handbook contains:

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

Interpretation:

- `data_sts2_windows_x86_64/` is the managed runtime payload.
- `SlayTheSpire2.pck` is the main Godot content pack.
- `mods/` is the obvious local mod install target.
- `tools/` exists in this install, but in this repository it is **local helper tooling**, not guaranteed vanilla STS2 content.

## Release Metadata

`release_info.json` currently reports:

```json
{
  "commit": "f4eeecc6",
  "version": "v0.98.2",
  "date": "2026-03-06T15:52:37-08:00",
  "branch": "v0.98.2"
}
```

Use this file whenever you need to annotate a guide with the exact build it was verified against.

## Managed Runtime Payload

The `data_sts2_windows_x86_64/` folder contains:

- `sts2.dll`
- `0Harmony.dll`
- `GodotSharp.dll`
- Steamworks and utility assemblies
- the .NET runtime assembly set used by the game

This is the directory you should reference when:

- decompiling the game
- resolving symbols in ILSpy or an IDE
- building a mod against the exact runtime surface the game expects

## What The Logs Confirm

From `sts2_stdout.log` on this machine:

- engine line: `MegaDot v4.5.1.m.8.mono.custom_build`
- release line: `release=v0.98.2`
- locale load examples:
  - `Loading locale path=res://localization/eng`
  - `Loading locale path=res://localization/rus`

This confirms three important facts:

1. STS2 is running a custom Mono-enabled Godot 4.5.1 build.
2. the game loads localization from Godot `res://` paths inside the main pack.
3. logs are useful for validating assumptions that decompiled code alone may not make obvious.

## What The Main `.pck` Confirms

Listing `SlayTheSpire2.pck` confirms:

- pack version: `3`
- Godot version: `4.5.1`
- vanilla localization files live under `localization/<lang>/...`
- vanilla relic image imports live under `images/relics/*.png.import`

Examples seen directly in the main pack:

- `localization/README.md`
- `localization/eng/relics.json`
- `localization/eng/cards.json`
- `localization/eng/characters.json`
- `localization/eng/static_hover_tips.json`
- `images/relics/akabeko.png.import`

## Vanilla vs Local Helper Tooling

This repository also contains helper tools under `tools/`, including:

- a PCK inspection tool
- ILSpy binaries
- local debug/loose-mod helpers

These are useful for research and development inside this repo, but you should not document them as if Mega Crit ships them to every player by default.

## Practical Takeaway

When in doubt:

1. check `release_info.json`
2. inspect `sts2.dll`
3. inspect `SlayTheSpire2.pck`
4. confirm with runtime logs

That order is more reliable than copying a tutorial verbatim.
