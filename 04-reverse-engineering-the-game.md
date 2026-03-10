# 04. Reverse-Engineering The Game

## Ground Rules

You are not working from the original Mega Crit source tree. For STS2 modding, "source verification" usually means:

- decompiled `sts2.dll`
- inspected `SlayTheSpire2.pck`
- runtime logs
- working example mods

That is normal for early-access modding.

## Start With The Right Targets

For modding-specific behavior, begin in these namespaces:

- `MegaCrit.Sts2.Core.Modding`
- `MegaCrit.Sts2.Core.Hooks`
- `MegaCrit.Sts2.Core.Models`
- `MegaCrit.Sts2.Core.Entities`
- `MegaCrit.Sts2.Core.Rooms`
- `MegaCrit.Sts2.Core.Combat`

For content-specific work:

- relics: inspect `Models.Relics`
- cards: inspect `Models.Cards`
- powers: inspect `Models.Powers`
- map/rewards: inspect room, reward, and pool systems

## Recommended Workflow

1. Identify the in-game object you want to modify.
2. Find its model type in `sts2.dll`.
3. Inspect methods, overrides, related pool classes, and any nearby hook or helper usage.
4. Search for the associated localization keys and asset names in `SlayTheSpire2.pck`.
5. Only after that decide whether you need:
   - a new model
   - a Harmony patch
   - a built-in hook
   - custom assets
   - localization additions

This order prevents a lot of unnecessary patching.

## Example: Relics

If you want to add or imitate a relic:

1. inspect a vanilla relic model in `sts2.dll`
2. inspect `localization/eng/relics.json`
3. inspect `images/relics/*.png.import`
4. inspect relic pool generation and model lookup code

The lamali guide uses Akabeko as a teaching example. That is a reasonable starting point because:

- `Akabeko` exists as a real vanilla relic model
- `akabeko.png.import` exists in the main pack
- the vanilla localization tables contain relic strings

## Use The Pack As A Second Source Of Truth

Decompiled code tells you behavior. The main pack tells you naming and content conventions.

Use pack inspection to answer questions like:

- what is the real asset folder called?
- is the file naming convention snake_case?
- which localization tables exist?
- what languages are supported?
- are these resources loaded from `res://` paths?

For STS2, that is often faster than guessing from class names.

## What To Look For In Modding Types

The current build exposes these especially useful entrypoints:

- `ModManager.LoadModsInDirRecursive`
- `ModManager.TryLoadModFromPck`
- `ModManager.CallModInitializer`
- `ModManager.GetModdedLocTables`
- `ModHelper.AddModelToPool`

Interpret them like this:

- mod loading is recursive, not flat
- `.pck` files are first-class load units
- initialization is attribute-driven
- mod localization tables are explicitly gathered
- there is helper support for adding models to pools

## Use Real Mods As Comparative Evidence

The unpacked example mods are useful for answering questions that decompiled game code alone does not answer cleanly:

- how authors actually package releases
- whether they ship source files inside `.pck`
- whether manifests are loose or embedded
- whether they include localization files
- whether a mod is code-only or asset-bearing

Example: `DamageMeter.pck` contains:

- embedded localization JSON
- many `.cs` files
- a `project.binary`
- `.godot` metadata

That is a real working example, not just a tutorial pattern.

## Avoid These Reverse-Engineering Mistakes

- Treating decompiler output as pretty source code instead of reconstructed behavior.
- Assuming the lamali folder layout is the only accepted one.
- Assuming a pattern from one mod generalizes to all mods.
- Ignoring runtime logs when they can confirm the actual path or version being used.

## Practical Checklist Before You Implement A Feature

Before you write code for a new mechanic, confirm:

1. the exact vanilla type you are targeting
2. whether a built-in STS2 hook already exists for it
3. how the corresponding assets are named
4. which localization table the vanilla content uses
5. whether a working example mod already demonstrates a similar packaging shape
