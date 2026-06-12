# 05. Behavior Mods, Harmony, And Hooks

## Explicit Initialization

The current loader searches a loaded assembly for types marked with `ModInitializerAttribute`. The attribute names a static method:

```csharp
using HarmonyLib;
using MegaCrit.Sts2.Core.Modding;

[ModInitializer(nameof(Initialize))]
public static class ModEntry
{
    public static void Initialize()
    {
        new Harmony("example.fieldnotes").PatchAll(typeof(ModEntry).Assembly);
    }
}
```

An explicit initializer is preferable even though `v0.103.3` falls back to automatic `Harmony.PatchAll` when no initializer exists. Registration, hook subscriptions, compatibility checks, and startup logging need a deterministic location.

## Three Integration Levels

Use the least invasive level that expresses the feature:

1. override a model method or use the command API when implementing a new model
2. use semantic hooks when behavior belongs to a named gameplay phase
3. use Harmony when no model or hook contract reaches the required behavior

This order separates extension from interception. A custom card should implement `OnPlay`; it should not patch the global card-play routine. A custom relic should override a hook such as `BeforeCombatStart`; it should not patch every combat constructor.

## Semantic Hooks

`MegaCrit.Sts2.Core.Hooks.Hook` exposes lifecycle and modifier hooks for combat, turns, cards, rooms, rewards, merchants, potions, powers, healing, damage, block, death prevention, and other systems.

Hook signatures are not uniform:

- notification hooks commonly return `Task`
- veto hooks commonly return `bool`
- modifier hooks return a transformed value
- many hooks receive `IRunState`, `CombatState`, `Player`, or `PlayerChoiceContext`

Read the exact signature and inspect at least one vanilla caller. The position of a hook in an async command chain determines whether visual cleanup and state mutation have already completed.

### Card Resolution

For an effect described as "after a card has resolved", use `AfterCardPlayedLate(PlayerChoiceContext, CardPlay)` as the baseline. It runs later than the immediate post-play hook and is safer for effects that assume result-pile movement and the normal play sequence have completed.

Do not replace or wrap a concrete card's `OnPlay` merely to observe completion. Owning the wrong task chain can leave cards in the wrong pile, interrupt animations, or duplicate cleanup.

## Mod-Owned Hook Subscribers

`v0.103.3` adds explicit subscription paths for models that are not naturally reached through a card, relic, power, or modifier collection:

```csharp
ModHelper.SubscribeForRunStateHooks(
    "example.fieldnotes.run",
    runState => GetRunScopedModels(runState));

ModHelper.SubscribeForCombatStateHooks(
    "example.fieldnotes.combat",
    combatState => GetCombatScopedModels(combatState));
```

Each delegate returns `IEnumerable<AbstractModel>`. Subscriber IDs must be unique; the game sorts them ordinally and ignores duplicate IDs after logging an error. Use this path for a persistent mod-owned rules object that must receive hooks but does not belong to a vanilla state collection.

## Harmony Patches

Harmony remains appropriate for:

- adding an event to a concrete act list
- inserting a launcher into an existing UI lifecycle
- correcting a value before no semantic hook exists
- observing a private implementation boundary after verifying its stability

Prefer exact targets:

```csharp
[HarmonyPatch(typeof(SomeType), nameof(SomeType.SomeMethod), typeof(Player))]
internal static class SomeMethodPatch
{
    private static void Postfix(Player player)
    {
        // Narrow behavior only.
    }
}
```

For overloaded methods, specify parameter types or use `AccessTools.Method` with an explicit signature. Never patch all methods sharing a name unless every overload is intentionally part of the feature.

## Description Methods

The current public card description entry point is:

```csharp
string GetDescriptionForPile(PileType pileType, Creature? target = null)
```

It delegates to a private preview-aware overload. Patch the public signature only if the feature truly changes rendered text, and verify that the patch is not also applied through a caller. Earlier advice to patch multiple `GetDescriptionForPile` overloads is obsolete and commonly duplicates text.

## Async Safety

When a target returns `Task`:

- use a model override or semantic hook if possible
- do not block with `.Wait()` or `.Result`
- preserve the original task unless intentionally replacing the operation
- keep gameplay commands in the game's async sequence

Commands such as `CreatureCmd`, `CardPileCmd`, `PlayerCmd`, and `RelicCmd` coordinate state, multiplayer choices, and visuals. Direct field mutation may bypass those systems.

## Patch Maintenance

For each Harmony patch, record:

- target type and exact signature
- why no model override or semantic hook is sufficient
- assumptions about call order and mutable state
- the game version last verified

After an update, re-decompile the target before launching the old DLL. A patch can still bind while its semantic position has changed.
