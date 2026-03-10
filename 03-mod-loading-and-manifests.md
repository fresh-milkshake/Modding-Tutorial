# 03. Mod Loading And Manifests

## Verified Modding Surface In `sts2.dll`

The following types and members are present in the current game build:

- `MegaCrit.Sts2.Core.Modding.ModManifest`
- `MegaCrit.Sts2.Core.Modding.ModManager`
- `MegaCrit.Sts2.Core.Modding.ModInitializerAttribute`
- `MegaCrit.Sts2.Core.Modding.Mod`
- `MegaCrit.Sts2.Core.Modding.ModSettings`
- `MegaCrit.Sts2.Core.Modding.ModSource`

Important `ModManager` members:

- `Initialize`
- `LoadModsInDirRecursive`
- `TryLoadModFromPck`
- `TryLoadModFromSteam`
- `CallModInitializer`
- `GetModdedLocTables`

That is enough to establish that STS2 has a built-in managed mod loader, supports recursive directory loading, and distinguishes between local mods and Steam Workshop mods.

## What A Manifest Looks Like

`ModManifest` contains these fields in the current build:

- `author`
- `description`
- `name`
- `pckName`
- `version`

Real example mods encode that data in JSON using keys such as:

```json
{
  "pck_name": "UpgradeAllCards",
  "name": "Upgrade All Cards",
  "author": "JiesiLuo",
  "description": "Automatically upgrades all starting cards and any new cards added to your deck during a run.",
  "version": "1.0.0"
}
```

## What Usually Ships In A Mod

Across the six example archives, the stable pattern is:

- a compiled `.dll`
- a `.pck`
- a `mod_manifest.json` either inside the `.pck`, outside it, or both

Optional extras seen in the wild:

- `README.txt`
- `CHANGELOG.txt`

## Packaging Patterns Observed In Real Mods

The example archives show at least three install layouts:

| Pattern | Example | Notes |
| --- | --- | --- |
| root-level `.dll + .pck` | `UpgradeAllCards`, `UndoAndRedo`, `typing`, `BetterSpire2` | simplest package shape |
| named subfolder containing payload | `BetterSovereignBlade` | archive extracts into a mod-named folder |
| `mods/<ModName>/...` already included | `DamageMeter` | installer-friendly archive; can often be copied directly into the game root |

This matters because community guides often imply one canonical layout. STS2 clearly tolerates more than one archive packaging style.

## Where The Manifest Can Live

Observed patterns:

- **inside `.pck` only:** `UpgradeAllCards`, `UndoAndRedo`, `BetterSpire2`, `typing`, `BetterSovereignBlade`
- **inside `.pck` and also loose next to payload:** `DamageMeter`

Do not assume a loose `mod_manifest.json` is mandatory just because one working mod includes it.

## Practical Meaning Of `pck_name`

The manifest field `pck_name` is not cosmetic. Treat it as an identity field tied to the mod payload name.

Safe rule:

- keep `pck_name`, the payload basename, and the logical mod name aligned unless you have a verified reason not to

Examples:

- `UpgradeAllCards.pck` uses `pck_name: "UpgradeAllCards"`
- `DamageMeter.pck` uses `pck_name: "DamageMeter"`

## Local Mods vs Workshop Mods

`ModSource` currently contains:

- `None`
- `ModsDirectory`
- `SteamWorkshop`

That proves the loader understands at least two sources:

- local folder mods under the game install
- Workshop-provided mods

This handbook focuses on local development and installation, but you should be aware that the loader itself already models Steam Workshop as a first-class source.

## There Is A Consent Gate

`ModManager` and `ModSettings` expose:

- `PlayerAgreedToModLoading`
- disabled mod tracking

This means STS2's loader is not just "drop files and hope"; there is user-facing state around mod loading and mod disablement.

## What To Put In Your Release Zip

For a straightforward local mod release, ship one of these:

### Option A: root payload

```text
MyMod.dll
MyMod.pck
```

### Option B: folder payload

```text
MyMod/
  MyMod.dll
  MyMod.pck
```

### Option C: installer-friendly root-ready package

```text
mods/
  MyMod/
    MyMod.dll
    MyMod.pck
    mod_manifest.json
```

`DamageMeter` is the cleanest real example of option C.

## Recommended Default

For a new mod, the lowest-friction release shape is:

```text
mods/
  MyMod/
    MyMod.dll
    MyMod.pck
    mod_manifest.json
```

Why this is the safest recommendation:

- it matches a working real-world example
- it avoids ambiguity about where the files belong
- it makes installation instructions trivial for users

It is still a recommendation, not the only working layout STS2 can load.
