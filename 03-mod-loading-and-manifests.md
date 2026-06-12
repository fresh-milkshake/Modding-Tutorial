# 03. Project Skeleton, Build, And Manifest

## Recommended Repository Shape

```text
FieldNotes/
  FieldNotes.csproj
  ModEntry.cs
  Cards/
  Potions/
  Relics/
  Events/
  Gui/
  Patches/
  godot/
    FieldNotes/
      localization/
      images/
    images/
      atlases/
  build/
    mods/
      FieldNotes/
```

The source tree separates managed models from Godot resources. The staging directory mirrors the final installation and should contain only files that will be shipped.

## Project File

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
           Text="Set GameDir or Sts2DataDir to the current Slay the Spire 2 installation." />
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

`Private=false` prevents copies of game-owned assemblies from being staged beside the mod DLL. The running game already supplies them.

## One External Manifest

`v0.103.3` uses one manifest file discovered below `<STS2>/mods`. Name it `<ModId>.json` and place it beside the declared payloads:

```json
{
  "id": "FieldNotes",
  "name": "Field Notes",
  "author": "Example Author",
  "description": "Adds an example card, potion, relic, event, and overlay.",
  "version": "0.1.0",
  "has_pck": true,
  "has_dll": true,
  "dependencies": [],
  "affects_gameplay": true
}
```

Current fields:

| Field | Meaning |
| --- | --- |
| `id` | Required loader identity and payload basename |
| `name` | Display name |
| `author` | Display metadata and fallback Harmony ID prefix |
| `description` | Display metadata |
| `version` | Mod version shown to the loader and metrics |
| `has_pck` | Require and load `<id>.pck` |
| `has_dll` | Require and load `<id>.dll` |
| `dependencies` | Manifest IDs that must load first |
| `affects_gameplay` | Whether the mod is included in gameplay-relevant mod reporting |

There is no `pck_name` field in the current `ModManifest`, and an embedded `mod_manifest.json` is not the active loader contract.

## Identity Contract

For a manifest with `"id": "FieldNotes"`, the loader resolves:

```text
<manifest directory>/FieldNotes.dll
<manifest directory>/FieldNotes.pck
```

The external manifest filename does not technically define identity, but matching it to the ID removes ambiguity and avoids accidental parsing of unrelated JSON files. Because every `.json` below `mods/` is treated as a possible manifest, do not store settings or arbitrary JSON beside installed mods unless they are outside the scanned tree or use a different extension.

## Initializer And Registration

Use an explicit initializer when the mod registers content, subscribes hooks, or needs startup logging:

```csharp
using HarmonyLib;
using MegaCrit.Sts2.Core.Modding;
using MegaCrit.Sts2.Core.Models.CardPools;
using MegaCrit.Sts2.Core.Models.PotionPools;
using MegaCrit.Sts2.Core.Models.RelicPools;

[ModInitializer(nameof(Initialize))]
public static class ModEntry
{
    public static void Initialize()
    {
        ModHelper.AddModelToPool<ColorlessCardPool, FieldNotesCard>();
        ModHelper.AddModelToPool<SharedPotionPool, BottledInsightPotion>();
        ModHelper.AddModelToPool<SharedRelicPool, SurveyorLensRelic>();

        new Harmony("example.fieldnotes").PatchAll(typeof(ModEntry).Assembly);
    }
}
```

Pool registration must happen before the game freezes pool contents. `ModHelper` throws when a model is added after initialization.

## Build And Stage

The expected output is:

```text
build/mods/FieldNotes/
  FieldNotes.json
  FieldNotes.dll
  FieldNotes.pck
```

A code-only mod sets `has_pck` to `false` and omits the PCK. A data-only pack sets `has_dll` to `false`, although most custom model examples require a DLL.

Build scripts should fail when a declared payload is absent. Silent partial staging produces manifests that the loader can detect but cannot satisfy.

## Installation

Copy the whole staged directory to:

```text
<STS2>/mods/FieldNotes/
```

Root-level manifests and payloads still work because discovery is recursive from `mods/`, but per-mod directories isolate filenames, simplify removal, and match the loader's directory ownership model.
