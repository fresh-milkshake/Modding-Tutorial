# 04. Reverse-Engineering The Game

## Ground Rules

For STS2 modding, reliable evidence usually means:

- decompiled `sts2.dll`
- inspected `SlayTheSpire2.pck`
- runtime logs
- observed working mod payloads

That is normal for an early-access game, and it changes how reverse engineering should be understood. The goal is not to recover an idealized source tree. The goal is to recover enough truthful structure to make correct integration decisions. In practice, that means distinguishing behavioral truth from cosmetic decompiler output, and distinguishing live shipped conventions from assumptions inherited from other Godot or C# projects.

## Start In The Right Namespaces

For modding-specific work, begin with:

- `MegaCrit.Sts2.Core.Modding`
- `MegaCrit.Sts2.Core.Hooks`
- `MegaCrit.Sts2.Core.Models`
- `MegaCrit.Sts2.Core.Entities`
- `MegaCrit.Sts2.Core.Rooms`
- `MegaCrit.Sts2.Core.Combat`

These namespaces are useful because they correspond to the conceptual boundaries a modder usually cares about: loading, gameplay events, model definitions, combat logic, and room or run flow. Starting there reduces the temptation to patch whatever method happens to be easiest to find first.

For content-specific work, inspect the matching model namespace first. A content mod that begins from generation code instead of the model contract often becomes more brittle than necessary.

## Recommended Workflow

1. identify the gameplay object or screen you want to change
2. find its real type in `sts2.dll`
3. inspect overrides, nearby helpers, pools, and hooks
4. inspect the matching assets and localization in `SlayTheSpire2.pck`
5. only then choose between a hook, a new model, a Harmony patch, assets, or new strings

This order matters because it forces problem definition before implementation. Many failed STS2 modding attempts are really classification errors: a modder thinks they need a Harmony patch, but the job is actually model registration; or they think the problem is code, but the real issue is a wrong `res://` path inside the pack. By walking from the gameplay target to the live type, and only then to code or content changes, you reduce that class of mistake.

## Use The Pack As A Second Source Of Truth

Decompiled code tells you behavior. The main pack tells you naming and content conventions.

Use `SlayTheSpire2.pck` to answer:

- real asset folder names
- language codes
- localization table names
- image naming conventions
- exact `res://` paths

This split is one of the most useful habits in STS2 reverse engineering. `sts2.dll` can tell you that a relic expects a large icon path or that localization is merged through a certain method. `SlayTheSpire2.pck` can tell you how that expectation is expressed in the shipped game, including actual filenames and table names. Neither source is complete on its own.

## Treat Preview Objects Carefully

Preview and inspect flows can use objects that are not identical to the source object you expect.

Example rule for upgrade work:

- confirm whether the preview object is already at the target state before applying more math

Do not assume the preview instance is always the original source instance. In UI-heavy systems, STS2 can present objects that are already transformed for display or preview purposes. If you patch these as though they were the underlying source objects, your logic can drift by a full step immediately.

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

These methods are useful not only because they exist, but because they reveal the game's preferred integration style. Recursive loading tells you to think in terms of payload discovery rather than a single hardcoded folder. Explicit localization merging tells you that string placement is governed by a known rule. `AddModelToPool` tells you that at least some content-extension paths are expected by the game rather than merely tolerated by it.

## Optional Helper Tooling

Local debug-hook and reflection helpers can speed up research, especially for UI work, but they are optional accelerators. Their role is to shorten the path from question to evidence, not to replace the live game as the source of truth. Keep the tutorial grounded in the live game, not in helper-tool assumptions.

## Practical Checklist Before You Implement

Before you add a feature, confirm:

1. the exact vanilla type you are targeting
2. whether a built-in hook already exists
3. which localization table the vanilla feature uses
4. which asset names and paths the vanilla feature expects
5. whether the job is really content registration, behavior interception, or UI reuse

This checklist is simple, but it captures the most common points where STS2 mod work goes wrong. A correct answer to all five questions usually narrows the implementation space dramatically.
