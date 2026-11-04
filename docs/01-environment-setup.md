# 01. Environment Setup

STS2 modding combines a managed assembly, a Godot resource pack, and a loader built into the game. The correct toolchain is therefore determined by the installed release rather than by a standalone mod SDK.

## Verified Toolchain

The baseline used throughout this handbook is:

- STS2 `v0.107.1`, commit `59260271`
- .NET target framework `net9.0`
- bundled Microsoft.NETCore.App `9.0.7`
- Godot PCK version `4.5.1`
- the `sts2.dll`, `0Harmony.dll`, and `GodotSharp.dll` files shipped with the same game install

`sts2.runtimeconfig.json` is the direct source for the target framework and runtime version. `release_info.json` is the direct source for the game release. Do not infer either value from an old project file.

## Required Files

For the default Steam installation, verify:

```text
<STS2>/
  release_info.json
  SlayTheSpire2.pck
  data_sts2_windows_x86_64/
    sts2.dll
    sts2.runtimeconfig.json
    0Harmony.dll
    GodotSharp.dll
```

The game installation acts as the SDK:

- `sts2.dll` defines models, commands, hooks, nodes, and loader types
- `0Harmony.dll` supplies the patching runtime already used by the game
- `GodotSharp.dll` supplies the exact Godot C# API loaded by the executable
- `SlayTheSpire2.pck` supplies resource names, localization tables, scenes, and imported asset conventions

Mixing assemblies from another STS2 version can compile successfully and still fail when a method, field, or generated Godot binding has changed.

## Create The Project

```powershell
dotnet new classlib -n FieldNotes -f net9.0
```

Keep machine-specific paths outside committed project logic when possible. A practical project accepts `GameDir` or `Sts2DataDir` as an MSBuild property and provides the common Steam path only as a local fallback.

## Tools

Required:

- .NET 9 SDK
- a C# editor
- a .NET decompiler such as ILSpy

Required only for content packs:

- a Godot 4.5.1 .NET editor or a compatible PCK packer

Useful local tools can be placed under `<STS2>/tools`, including `godotpcktool.exe`. The inspected installation has that executable, but its presence does not establish that every vanilla Steam install includes it. Treat it as an optional development tool, not a mod runtime dependency.

## Preflight Check

Before writing code:

1. read the current version from `release_info.json`
2. read the TFM from `sts2.runtimeconfig.json`
3. confirm all three referenced DLLs come from the same data directory
4. inspect the PCK header and confirm its Godot version
5. launch the unmodified game once and locate its current logs

Record these values in release notes or a maintenance issue. Version evidence makes later breakage reproducible.

## Common Setup Failures

- targeting the latest installed .NET SDK instead of the game's TFM
- using NuGet Harmony or GodotSharp assemblies that differ from the shipped copies
- decompiling an old backup of `sts2.dll`
- packing assets with an incompatible Godot version
- assuming a successful DLL build proves the PCK and manifest are valid

The next chapter maps these files to the loader stages that consume them.

## Verification And Branches

Last API and resource inspection: **2026-11-04**, stable `v0.107.1` (`59260271`). This baseline describes the installed stable branch. The public beta is a separate compatibility target; do not assume its APIs or balance changes are present here.

The code examples are checked by compilation against the installed assemblies. Compilation and resource inspection do not replace the in-game validation matrices in each chapter.
