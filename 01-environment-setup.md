# 01. Environment Setup

## Required Baseline

For the current local STS2 build used in this handbook, the practical toolchain is:

- Slay the Spire 2 installed via Steam
- .NET 9 SDK
- Godot 4.5.1 .NET
- a C# IDE or decompiler

Why this exact stack:

- `sts2.dll` is a managed assembly
- the game ships `0Harmony.dll` and `GodotSharp.dll`
- the main game pack reports Godot `4.5.1`
- `sts2_stdout.log` reports `MegaDot v4.5.1.m.8.mono.custom_build`

Use the game's own assemblies and the matching Godot line. Do not build against random copies.

## Baseline Verified Here

- STS2 release version: `v0.99.1`
- release date: `2026-03-13T20:40:28-07:00`
- engine line: `MegaDot v4.5.1.m.8.mono.custom_build`
- main pack Godot version: `4.5.1`

## Minimum Local Checks

Before starting a mod, verify:

1. `release_info.json` exists in the game root
2. `data_sts2_windows_x86_64\sts2.dll` exists
3. `data_sts2_windows_x86_64\0Harmony.dll` exists
4. `data_sts2_windows_x86_64\GodotSharp.dll` exists
5. `SlayTheSpire2.pck` exists
6. `mods\` exists or can be created

Optional but useful:

1. launch the game once
2. inspect `sts2_stdout.log`
3. confirm the engine and release lines

## Minimal Bootstrap

Create a starter library:

```powershell
dotnet new classlib -n MyMod -f net9.0
```

Keep machine-specific paths out of source control. Use local-only props, environment variables, or user-overridden MSBuild properties for:

- game install path
- Godot executable path
- optional local deploy path

## Common Setup Mistakes

- using standard Godot instead of the .NET build
- referencing assemblies from a different STS2 version than the one being tested
- copying a template without checking the current game install
- treating `.dll` output as the full mod when the mod also needs a `.pck`
