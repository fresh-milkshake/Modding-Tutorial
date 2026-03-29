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

This entry model is important because it shows that STS2 does not expect a special assembly name or a convention-based top-level bootstrap class. Instead, it expects a type marked as an initializer owner, and it expects that type to expose a callable method whose name matches the attribute argument. That is a narrow but clear contract, and it is one of the simplest places to diagnose packaging failures that otherwise look mysterious.

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

Hooks are valuable not only because they are convenient, but because they encode the game's own understanding of its event boundaries. If STS2 already distinguishes `AfterCombatEnd` from `AfterCombatVictory`, or `AfterCardPlayed` from a later post-resolution path, that distinction is information. Using the hook preserves the game's semantics instead of forcing the mod to infer them from lower-level implementation details.

## Use Harmony When Needed

Use Harmony when:

- there is no built-in hook for the behavior
- you need a precise method interception
- you are changing implementation details rather than registering content

Read the exact signature before patching. STS2 uses `Task`, `bool`, and modifier-return patterns across both hooks and ordinary methods. A patch that ignores whether a method is part of an async chain, a veto path, or a value-transform path can appear correct at compile time while still breaking runtime ownership and sequencing.

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

This is one of the places where STS2 rewards patience. A patch placed too early in the play lifecycle may still produce the visible gameplay effect you wanted, which makes it tempting to declare success. The real regression often appears later, when a card remains on screen too long, moves to the wrong pile, or leaves UI state behind. In other words, "the effect fired" is not the same as "the integration point was correct."

## Description Patch Warning

`CardModel.GetDescriptionForPile` has multiple overloads in the current local assembly.

Safe rule:

- patch the deepest overload you actually need
- do not blindly patch all matching names

Otherwise it is easy to append the same custom text twice, especially if one overload delegates into another. Text rendering paths often look simpler than they are because the visible output is short while the call chain is layered.

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

This incremental style is not just about cleanliness. It keeps the mod legible to its future maintainer, and in an Early Access environment that legibility is a practical defense against update churn.
