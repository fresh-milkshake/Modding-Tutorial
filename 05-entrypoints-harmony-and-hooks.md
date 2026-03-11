# 05. Entrypoints, Harmony, And Hooks

## The Core STS2 Entry Mechanism

The current build contains `MegaCrit.Sts2.Core.Modding.ModInitializerAttribute` with a constructor that takes a string initializer method name.

Real example mods use it like this at the **type** level:

- `UpgradeAllCards.UpgradeAllCardsMod => ModInitializer(Initialize)`
- `BetterSpire2.ModEntry => ModInitializer(Init)`
- `DamageMeter.MainFile => ModInitializer(Initialize)`

That means the default STS2-native entry pattern is:

```csharp
using HarmonyLib;
using MegaCrit.Sts2.Core.Modding;

[ModInitializer("Initialize")]
public static class ModEntry
{
    public static void Initialize()
    {
        var harmony = new Harmony("com.example.mymod");
        harmony.PatchAll();
    }
}
```

This pattern is consistent with both working example mods and the lamali guide.

## Why The Attribute Is On The Type

`CallModInitializer` in `ModManager` strongly suggests STS2 scans mod assemblies for attributed types and then invokes the named method. That is why the string method name matters.

Practical implications:

- the method name in the attribute must exist
- the method must be callable at mod startup
- a typo here is enough to make your mod appear packaged correctly but fail to initialize

## Harmony Is The Default Escape Hatch

Use Harmony when:

- STS2 does not expose a hook for what you need
- you need to alter a very specific method body or return value
- you are changing behavior inside an existing code path rather than registering new content

Starter skeleton:

```csharp
using HarmonyLib;

[HarmonyPatch(typeof(SomeType), nameof(SomeType.SomeMethod))]
public static class SomeMethodPatch
{
    public static void Prefix()
    {
    }

    public static void Postfix()
    {
    }
}
```

## Prefer Hooks When A Hook Already Describes The Event

The current build contains a large built-in hook surface under:

```text
MegaCrit.Sts2.Core.Hooks.Hook
```

If an STS2 hook already matches your desired gameplay event, prefer it over a brittle Harmony patch.

Why:

- hooks are semantically named
- hooks tend to survive internal refactors better than patching arbitrary implementation methods
- hooks communicate intent better when other modders read your code

## Verified Hook Categories

The hook surface in the current build covers all of these areas:

### Combat lifecycle

- `BeforeCombatStart`
- `AfterCombatEnd`
- `AfterCombatVictory`
- `ShouldStopCombatFromEnding`

### Turn flow

- `AfterPlayerTurnStart`
- `AfterSideTurnStart`
- `BeforeTurnEnd`
- `AfterTurnEnd`
- `ShouldTakeExtraTurn`
- `AfterTakingExtraTurn`

### Damage and block

- `BeforeAttack`
- `AfterAttack`
- `ModifyDamage`
- `ModifyDamageInternal`
- `AfterDamageGiven`
- `AfterDamageReceived`
- `ModifyBlock`
- `BeforeBlockGained`
- `AfterBlockGained`
- `ShouldClearBlock`

### Cards and piles

- `BeforeCardPlayed`
- `AfterCardPlayed`
- `BeforeCardAutoPlayed`
- `AfterCardDrawn`
- `AfterCardDiscarded`
- `AfterCardExhausted`
- `AfterCardChangedPiles`
- `ModifyCardPlayCount`
- `ModifyCardRewardOptions`

### Rewards, merchants, economy

- `BeforeRewardsOffered`
- `AfterRewardTaken`
- `ModifyRewards`
- `ModifyMerchantPrice`
- `ModifyMerchantCardPool`
- `ShouldGainGold`
- `AfterGoldGained`
- `ShouldGainStars`
- `AfterStarsGained`

### Rooms, maps, acts

- `BeforeRoomEntered`
- `AfterRoomEntered`
- `AfterActEntered`
- `AfterMapGenerated`
- `ModifyGeneratedMap`
- `ModifyNextEvent`

### Potions, powers, healing

- `BeforePotionUsed`
- `AfterPotionUsed`
- `AfterPotionProcured`
- `ModifyPowerAmountGiven`
- `ModifyPowerAmountReceived`
- `ModifyHealAmount`
- `ModifyRestSiteHealAmount`

### Death prevention and terminal checks

- `BeforeDeath`
- `AfterDeath`
- `ShouldDie`
- `AfterPreventingDeath`

## Hook Return Shapes Matter

The current API uses several different hook styles:

- `Task` for asynchronous or post-event flows
- `bool` for allow/deny decisions
- modifier-style methods for value transformation

Do not assume every hook is fire-and-forget. Read the exact signature before implementing one.

## Harmony vs Hook Decision Rule

Use this rule:

- if the game already names the event you care about in `Hook`, start there
- if you need a precise implementation interception that has no hook, use Harmony
- if you are adding new models or content pools, inspect `ModHelper` and related pool code before you patch anything

## Two Common STS2 Mod Shapes

### Behavior mod

Use this when you are changing existing game behavior without adding brand-new content.

Typical structure:

1. `[ModInitializer]` entry class
2. Harmony setup
3. a few focused patches
4. optional logging

Examples from the archive set:

- `UpgradeAllCards`
- `UndoAndRedo`
- `BetterSpire2`

### Content mod

Use this when you are adding new game content such as relic-like or card-like data that must become discoverable by STS2 systems.

Typical structure:

1. `[ModInitializer]` entry class
2. content model classes
3. registration or pool-extension logic
4. assets and localization in the `.pck`
5. optional Harmony patches only where registration alone is insufficient

For this path, inspect:

- the relevant vanilla model class
- the relevant pool generation code
- `ModHelper.AddModelToPool`
- any existing hook surface that can avoid a brittle patch

The lamali guide demonstrates this path using a custom relic workflow. That is a good public starting pattern, but validate the exact registration path against the current STS2 build before scaling it up.

## Custom Relic Contracts Worth Verifying

For relic mods in the current build, these `RelicModel` contracts are especially important:

- `Title`, `Description`, and `Flavor` default to `base.Id.Entry + ".title|description|flavor"` in the `relics` loc table
- `IconBaseName` defaults to `Id.Entry.ToLowerInvariant()`
- `PackedIconPath` and `PackedIconOutlinePath` point at atlas entries based on `IconBaseName`
- `BigIconPath` points at `res://images/relics/<IconBaseName>.png`

Practical consequence:

- a relic can show the expected small icon while still showing the wrong or missing large inspect art if `IconBaseName` does not match the actual file you packed

There is one more hidden dependency:

- `DynamicDescription` uses `EnergyIconHelper.GetPrefix(this)`
- for relics, that path calls `RelicModel.Pool`
- `RelicModel.Pool` expects the relic to belong to at least one `RelicPool`

If your custom relic is start-only or otherwise excluded from normal pool generation, hover or inspect screens can still throw when they try to render `DynamicDescription`.

Safe rule:

- either make the relic discoverable from an appropriate `RelicPool`, or patch the getter path so UI code can still resolve a pool for formatting and inspect screens

## Practical Entry Strategy For New Mods

For a small STS2 mod, start with:

1. one `[ModInitializer]` entry class
2. one Harmony instance with a stable unique ID
3. a small number of focused patch classes
4. hooks where the semantic event already exists

Avoid beginning with a large patch-all architecture before you know which integration points are stable.
