# 05. Behavior Mods, Harmony, And Hooks

## Core Entry Mechanism

The current build exposes `ModInitializerAttribute` with a string method name. The standard STS2 entry pattern is type-level:

```csharp
using HarmonyLib;
using MegaCrit.Sts2.Core.Modding;

[ModInitializer("Initialize")]
public static class ModEntry
{
    public static void Initialize()
    {
        var harmony = new Harmony("com.example.mymod");
        harmony.PatchAll(typeof(ModEntry).Assembly);
    }
}
```

Practical implications:

- the attribute belongs on a type
- the named method must exist
- a typo can leave a correctly packaged mod unloaded at runtime

## Use Hooks First

The built-in hook surface under `MegaCrit.Sts2.Core.Hooks.Hook` covers:

- combat lifecycle
- turn flow
- damage and block
- cards and piles
- rewards, merchants, and economy
- rooms, map flow, and acts
- potions, powers, and healing
- death prevention and combat-ending checks

If the game already names the event you need, prefer the hook over a Harmony patch.

Why:

- hooks express intent directly
- hooks tend to survive internal refactors better
- hooks reduce the amount of method-body interception you own

## Use Harmony When Needed

Use Harmony when:

- there is no built-in hook for the behavior
- you need a precise method interception
- you are changing implementation details rather than registering content

Read the exact signature before patching. STS2 uses `Task`, `bool`, and modifier-return patterns across both hooks and ordinary methods.

## Safe Hook Selection Matters

For card behavior, not every patch point that "works" is equally safe.

Prefer `AfterCardPlayedLate(PlayerChoiceContext, CardPlay)` when your effect is conceptually:

- after full card resolution
- after discard or result-pile movement
- after the game's own animation and cleanup work

Be careful when intercepting or replacing concrete `OnPlay(...)` task chains. That can break:

- discard/result-pile movement
- animation cleanup
- card UI state

## Description Patch Warning

`CardModel.GetDescriptionForPile` has multiple overloads in the current local assembly.

Safe rule:

- patch the deepest overload you actually need
- do not blindly patch all matching names

Otherwise it is easy to append the same custom text twice.

## Practical Decision Rule

Use this order:

1. built-in hook if the game already names the event
2. `ModHelper` or registration path if the job is content
3. Harmony only when neither of the above covers the requirement

Start small:

1. one `[ModInitializer]` entry point
2. one Harmony instance with a stable unique ID
3. a small number of focused patches
4. explicit logging around initialization and patch registration
