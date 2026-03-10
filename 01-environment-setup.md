# 01. Environment Setup

## What You Actually Need

For the current STS2 baseline used in this handbook, the practical setup is:

- **Slay the Spire 2 installed via Steam**
- **.NET 9 SDK**
- **Godot 4.5.1 .NET**
- **a C# IDE or decompiler**
- **Harmony knowledge and access to Harmony assemblies through the game install**

Why this exact stack:

- `sts2.dll` is a managed .NET assembly.
- the game install includes `0Harmony.dll`, `GodotSharp.dll`, and .NET 9 runtime assemblies under `data_sts2_windows_x86_64`.
- `SlayTheSpire2.pck` reports Godot `4.5.1`.
- `sts2_stdout.log` reports `MegaDot v4.5.1.m.8.mono.custom_build`.

That is why the safest default is to build against the game's own managed assemblies and to package assets with **Godot 4.5.1 .NET**, not a random nearby version.

## Recommended Toolchain

Primary tools:

- [Godot 4.5.1 .NET archive](https://godotengine.org/download/archive/4.5.1-stable/)
- [.NET 9 SDK](https://dotnet.microsoft.com/download/dotnet/9.0)
- [Visual Studio](https://visualstudio.microsoft.com/) or Rider
- [ILSpy](https://github.com/icsharpcode/ILSpy)
- [Harmony docs](https://harmony.pardeike.net/articles/intro.html)

Community STS2 reference:

- [lamali292 STS2 C# Modding Guide](https://lamali292.github.io/sts2_modding_guide/)

## Minimum Folder Knowledge

On this machine, the game root is:

```text
C:\Program Files (x86)\Steam\steamapps\common\Slay the Spire 2
```

The most important subdirectory for mod development is:

```text
data_sts2_windows_x86_64
```

This is where the managed assemblies live:

- `sts2.dll`
- `0Harmony.dll`
- `GodotSharp.dll`
- the .NET runtime assemblies the game actually uses

## Baseline Verification Checklist

Before starting a mod project, verify these facts locally:

1. `release_info.json` exists in the game root.
2. `data_sts2_windows_x86_64\sts2.dll` exists.
3. `data_sts2_windows_x86_64\0Harmony.dll` exists.
4. `data_sts2_windows_x86_64\GodotSharp.dll` exists.
5. `SlayTheSpire2.pck` exists in the game root.
6. `mods\` exists or can be created under the game root.

Optional but useful:

1. launch the game once
2. inspect `sts2_stdout.log`
3. confirm the engine line and locale load lines

## Community Guide vs Verified Baseline

The lamali guide recommends:

- Godot 4.5.1 .NET
- .NET 9
- a class library project
- `[ModInitializer]`-based entrypoints

Those recommendations align with the current local install and with working example mods. That is why this handbook adopts them as the default path.

## Strong Recommendation On Version Matching

Example mods in `EXAMPLE_MODS` show mixed `.pck` pack versions:

- some `.pck` files were created with Godot `4.5.1`
- one example pack was created with Godot `4.3.0`

That proves older packs can sometimes still load. It does **not** make older packers the recommended choice. The safest baseline is still:

- compile against the current game's assemblies
- package assets with the same Godot major/minor line the game is using

## Suggested Project Skeleton

You do not need a complicated repo structure for a small STS2 mod. A practical starting shape is:

```text
MyMod/
  MyMod.csproj
  ModEntry.cs
  Patches/
  godot/
```

The exact Godot-side contents are covered later in [06-assets-pck-and-localization.md](06-assets-pck-and-localization.md).

## Minimal Project Bootstrap

Create a starter library:

```powershell
dotnet new classlib -n MyMod -f net9.0
```

Then point the project at the **current game's own assemblies**, not random NuGet packages or copies from another install.

Minimal `.csproj` sketch:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <LangVersion>latest</LangVersion>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <GameDir>C:\Program Files (x86)\Steam\steamapps\common\Slay the Spire 2</GameDir>
    <Sts2DataDir>$(GameDir)\data_sts2_windows_x86_64</Sts2DataDir>
  </PropertyGroup>

  <ItemGroup>
    <Reference Include="sts2">
      <HintPath>$(Sts2DataDir)\sts2.dll</HintPath>
    </Reference>
    <Reference Include="0Harmony">
      <HintPath>$(Sts2DataDir)\0Harmony.dll</HintPath>
    </Reference>
    <Reference Include="GodotSharp">
      <HintPath>$(Sts2DataDir)\GodotSharp.dll</HintPath>
    </Reference>
  </ItemGroup>
</Project>
```

This is intentionally minimal. Add more references only when the compiler proves you need them.

## Keep Machine-Specific Paths Out Of Source Control

The lamali guide uses a `local.props` pattern. That is a good idea even if your exact file shape differs.

Use some local-only mechanism for:

- game install path
- Godot executable path
- optional deployment output path

Do not hardcode your own absolute paths into a project you plan to share.

## Common Setup Mistakes

- Using standard Godot instead of the **.NET** build.
- Referencing assemblies from a different STS2 version than the one you are testing against.
- Treating a community template as authoritative without checking the current game install.
- Building only a `.dll` and forgetting that assets/localization usually travel in a `.pck`.
