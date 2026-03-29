# 04. Reverse-Engineering The Game

## Ground Rules

For STS2 modding, reliable evidence usually means:

- decompiled `sts2.dll`
- inspected `SlayTheSpire2.pck`
- runtime logs
- observed working mod payloads

That is normal for an early-access game.

## Start In The Right Namespaces

For modding-specific work, begin with:

- `MegaCrit.Sts2.Core.Modding`
- `MegaCrit.Sts2.Core.Hooks`
- `MegaCrit.Sts2.Core.Models`
- `MegaCrit.Sts2.Core.Entities`
- `MegaCrit.Sts2.Core.Rooms`
- `MegaCrit.Sts2.Core.Combat`

For content-specific work, inspect the matching model namespace first.

## Recommended Workflow

1. identify the gameplay object or screen you want to change
2. find its real type in `sts2.dll`
3. inspect overrides, nearby helpers, pools, and hooks
4. inspect the matching assets and localization in `SlayTheSpire2.pck`
5. only then choose between a hook, a new model, a Harmony patch, assets, or new strings

This order avoids a lot of unnecessary patching.

## Use The Pack As A Second Source Of Truth

Decompiled code tells you behavior. The main pack tells you naming and content conventions.

Use `SlayTheSpire2.pck` to answer:

- real asset folder names
- language codes
- localization table names
- image naming conventions
- exact `res://` paths

## Treat Preview Objects Carefully

Preview and inspect flows can use objects that are not identical to the source object you expect.

Example rule for upgrade work:

- confirm whether the preview object is already at the target state before applying more math

Do not assume the preview instance is always the original source instance.

## High-Value Entry Points

These current members are especially useful when reasoning about STS2 modding:

- `ModManager.LoadModsInDirRecursive`
- `ModManager.TryLoadModFromPck`
- `ModManager.CallModInitializer`
- `ModManager.GetModdedLocTables`
- `ModHelper.AddModelToPool`

Interpretation:

- mod loading is recursive
- `.pck` is a first-class load unit
- initialization is attribute-driven
- modded localization is merged explicitly
- model registration has dedicated helper support

## Optional Helper Tooling

Local debug-hook and reflection helpers can speed up research, especially for UI work, but they are optional accelerators. Keep the tutorial grounded in the live game, not in helper-tool assumptions.

## Practical Checklist Before You Implement

Before you add a feature, confirm:

1. the exact vanilla type you are targeting
2. whether a built-in hook already exists
3. which localization table the vanilla feature uses
4. which asset names and paths the vanilla feature expects
5. whether the job is really content registration, behavior interception, or UI reuse
