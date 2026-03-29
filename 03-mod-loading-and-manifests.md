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

The stable local staging pattern is `build/mods/<ModId>/`.

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

Use it inside the `.pck`, in staging, or both. Do not copy loose `mod_manifest.json` files into the root `mods\` directory for installed mods.

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

Keep `id`, payload basenames, and `pck_name` aligned unless you have a verified reason not to.

## Build And Staging Flow

The common local flow is:

1. build the `.dll`
2. copy the `.dll` and `godot/mod_manifest.json` into `build/mods/<ModId>/`
3. pack the Godot-side payload into `<ModId>.pck`
4. write `<ModId>.json`
5. optionally copy the final root-level files into the game `mods\` directory

Useful local helper tools such as `godotpcktool.exe` can speed this up, but they are optional helpers, not a vanilla STS2 requirement.

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

Archive shape can vary. Installed shape should be normalized.
