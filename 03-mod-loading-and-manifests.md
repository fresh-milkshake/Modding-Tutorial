# 03. Project Skeleton, Build, And Manifests

## Recommended Project Shape

For a small STS2 mod, start with:

```text
MyMod/
  MyMod.csproj
  ModEntry.cs
  Patches/
  godot/
    mod_manifest.json
  build.ps1
  build/
    mods/
      MyMod/
```

This structure is minimal, but it captures the actual boundaries that matter. The C# side lives in the project root and `Patches/`, where managed code, entrypoints, and Harmony patches can remain visible and easy to inspect. The Godot-side payload lives under `godot/`, which makes it obvious which files are intended to be packed into the `.pck`. The `build/` directory becomes a staging area rather than a source directory, which keeps generated output separate from authored files.

The stable local staging pattern is `build/mods/<ModId>/`. That is useful both conceptually and operationally: it gives you a single place to inspect the exact payload you are about to install or ship.

## Minimal `.csproj`

Use the current game assemblies directly:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <LangVersion>latest</LangVersion>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <GameDir Condition="'$(GameDir)' == '' and Exists('C:\Program Files (x86)\Steam\steamapps\common\Slay the Spire 2\data_sts2_windows_x86_64\sts2.dll')">C:\Program Files (x86)\Steam\steamapps\common\Slay the Spire 2</GameDir>
    <Sts2DataDir Condition="'$(Sts2DataDir)' == '' and '$(GameDir)' != ''">$(GameDir)\data_sts2_windows_x86_64</Sts2DataDir>
  </PropertyGroup>

  <Target Name="ValidateGameReferences" BeforeTargets="ResolveReferences">
    <Error Condition="'$(Sts2DataDir)' == '' or !Exists('$(Sts2DataDir)\sts2.dll')"
           Text="Set GameDir or Sts2DataDir to your Slay the Spire 2 install before building." />
  </Target>

  <ItemGroup>
    <Reference Include="sts2">
      <HintPath>$(Sts2DataDir)\sts2.dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="0Harmony">
      <HintPath>$(Sts2DataDir)\0Harmony.dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="GodotSharp">
      <HintPath>$(Sts2DataDir)\GodotSharp.dll</HintPath>
      <Private>false</Private>
    </Reference>
  </ItemGroup>
</Project>
```

This project file is intentionally plain. The important part is not the amount of MSBuild logic but the direction of dependency flow. The mod depends on the installed game, not on an abstract package feed. `TargetFramework` should match the active runtime line used by the game, which in the current local baseline is `net9.0`. The guarded `GameDir` and `Sts2DataDir` properties make it possible to build on a machine with the common Steam layout while still failing early and clearly on machines that differ.

The explicit references also serve as documentation. They say, in concrete form, that a typical STS2 C# mod relies on the game assembly, Harmony, and GodotSharp, and that those references are expected to come from the game install itself.

## Two Manifest Layers

On the current local build, treat these as separate files with separate roles.

### Payload manifest: `godot/mod_manifest.json`

This is the STS2 payload contract. `ModManifest` currently maps these fields:

- `author`
- `description`
- `name`
- `pckName`
- `version`

On disk, the JSON key is `pck_name`.

This file describes the mod payload as STS2 understands it. It belongs to the `.pck` layer and should be thought of as part of the Godot-side package contract. In practice, authors often keep a copy in staging as well, because it is useful for inspection and build scripting, but its natural home is still the payload being packed.

The important operational rule is negative: do not copy loose `mod_manifest.json` files into the root `mods\` directory for installed mods. The filename is generic, so multiple installed mods would collide immediately at that level.

### Installed-mod record: `<ModId>.json`

On this machine, the installed-mod UI was most reliable with root-level metadata shaped like:

```json
{
  "id": "MyMod",
  "name": "My Mod",
  "author": "Author",
  "description": "Short description.",
  "version": "v0.1.0",
  "has_pck": true,
  "has_dll": true,
  "dependencies": [],
  "affects_gameplay": true
}
```

This file serves a different audience. It is not primarily about the internal contents of the Godot pack; it is about the installed-mod record seen by the current loader and UI path on the local build. The safest documentation strategy is therefore to describe it as a separate layer, not as a synonym for `mod_manifest.json`.

Keep `id`, payload basenames, and `pck_name` aligned unless you have a verified reason not to. In ordinary cases, identity drift between these names creates more confusion than flexibility.

## Build And Staging Flow

The common local flow is:

1. build the `.dll`
2. copy the `.dll` and `godot/mod_manifest.json` into `build/mods/<ModId>/`
3. pack the Godot-side payload into `<ModId>.pck`
4. write `<ModId>.json`
5. optionally copy the final root-level files into the game `mods\` directory

This sequence is worth understanding as a pipeline rather than a checklist. The `.dll` is the managed behavior layer. The `.pck` is the Godot-side content layer. `<ModId>.json` is installation metadata for the current local build. Staging gathers those layers into one inspectable directory before installation. If something breaks, knowing which stage produced the break is often the fastest path to a fix.

Useful local helper tools such as `godotpcktool.exe` can speed this up, but they are optional helpers, not a vanilla STS2 requirement. The tutorial should describe the packaging model first and the local convenience tooling second.

## Current Installed Layout Recommendation

For the local `v0.99.1` build used here, the safest installed shape is:

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

This recommendation is deliberately conservative. Real release archives can vary, and the game may tolerate more than one top-level zip layout, but installation is where ambiguity becomes expensive. A normalized installed shape gives the loader and installed-mod UI the clearest possible input and gives the author the simplest possible troubleshooting surface.

Archive shape can vary. Installed shape should be normalized.
