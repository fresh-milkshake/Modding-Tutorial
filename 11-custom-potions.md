# 11. Custom Potions

Potions resemble cards at the model and pool layers, but their targeting and art contracts are different. The example below adds `BottledInsightPotion`, a combat potion that lets any player draw three cards.

## Implementation

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Godot;
using MegaCrit.Sts2.Core.Commands;
using MegaCrit.Sts2.Core.Entities.Cards;
using MegaCrit.Sts2.Core.Entities.Creatures;
using MegaCrit.Sts2.Core.Entities.Potions;
using MegaCrit.Sts2.Core.GameActions.Multiplayer;
using MegaCrit.Sts2.Core.Localization.DynamicVars;
using MegaCrit.Sts2.Core.Models;
using MegaCrit.Sts2.Core.Nodes.Rooms;

namespace FieldNotes.Potions;

public sealed class BottledInsightPotion : PotionModel
{
    public override PotionRarity Rarity => PotionRarity.Common;

    public override PotionUsage Usage => PotionUsage.CombatOnly;

    public override TargetType TargetType => TargetType.AnyPlayer;

    protected override IEnumerable<DynamicVar> CanonicalVars =>
    [
        new CardsVar(3)
    ];

    protected override async Task OnUse(
        PlayerChoiceContext choiceContext,
        Creature? target)
    {
        AssertValidForTargetedPotion(target);

        NCombatRoom.Instance?.PlaySplashVfx(
            target,
            new Color("45e6d0"));

        var player = target.Player
            ?? throw new InvalidOperationException(
                "Potion target is not a player.");

        await CardPileCmd.Draw(
            choiceContext,
            DynamicVars.Cards.BaseValue,
            player);
    }
}
```

The three required properties answer different questions:

- `Rarity` controls generation and merchant value
- `Usage` controls where the potion may be consumed
- `TargetType` controls whether the game must ask for a creature or player target

For a targeted potion, call `AssertValidForTargetedPotion` before dereferencing the target. The game can then report an invalid integration instead of producing a less useful null-reference failure.

## Registration

```csharp
using FieldNotes.Potions;
using MegaCrit.Sts2.Core.Modding;
using MegaCrit.Sts2.Core.Models.PotionPools;

ModHelper.AddModelToPool<SharedPotionPool, BottledInsightPotion>();
```

`PotionModel.Pool` searches `ModelDb.AllPotionPools` for a pool containing the model ID. Pool membership is therefore required even if an event grants the potion directly.

Use a character potion pool only when the potion is meant to be character-specific. `SharedPotionPool` makes it part of the general potion population.

## Localization

Create `godot/FieldNotes/localization/eng/potions.json`:

```json
{
  "BOTTLED_INSIGHT_POTION.title": "Bottled Insight",
  "BOTTLED_INSIGHT_POTION.description": "Draw {Cards:diff()} cards.",
  "BOTTLED_INSIGHT_POTION.selectionScreenPrompt": "Choose a player to draw cards."
}
```

The selection prompt matters for target-driven use flows. Even if one mode currently skips the prompt, complete localization prevents a later UI path from exposing a raw key.

## Fixed Art Path

Unlike card portraits and relic icons, the current potion image path is not virtual. `PotionModel` resolves:

```text
res://images/atlases/potion_atlas.sprites/bottled_insight_potion.tres
```

The optional outline is:

```text
res://images/atlases/potion_outline_atlas.sprites/bottled_insight_potion.tres
```

Keep the source image namespaced, then add a small `AtlasTexture` bridge resource at the fixed path:

```text
godot/
  FieldNotes/
    images/potions/bottled_insight.png
  images/
    atlases/
      potion_atlas.sprites/
        bottled_insight_potion.tres
```

Example bridge resource:

```ini
[gd_resource type="AtlasTexture" load_steps=2 format=3]

[ext_resource type="Texture2D" path="res://FieldNotes/images/potions/bottled_insight.png" id="1_texture"]

[resource]
atlas = ExtResource("1_texture")
region = Rect2(0, 0, 64, 128)
```

Set the region to the actual source dimensions. The resource must load as `Texture2D`; placing only the PNG at a namespaced path does not satisfy the fixed `.tres` lookup.

The outline resource can reference a separate outline PNG. If it is absent, the current model returns no outline rather than failing the base image lookup.

## Usability Extensions

`PotionModel` also supports:

- `CanBeGeneratedInCombat`
- custom usability checks
- extra hover tips
- selection-screen prompts
- combat-only and broader usage modes

Use a custom usability check for state-dependent rules such as requiring a non-empty discard pile. Return a precise reason through the same pattern used by a nearby vanilla potion so disabled UI remains understandable.

## Multiplayer Targeting

`TargetType.AnyPlayer` is meaningful in multiplayer. The selected `Creature` identifies the target player through `target.Player`, while `PlayerChoiceContext` preserves the decision flow.

Do not replace this with the local player singleton unless the design is explicitly local-only. Doing so can make the potion appear to target another player while applying the effect to the user who clicked it.

## Validation Matrix

Verify:

1. `SharedPotionPool.AllPotions` contains the potion
2. normal rewards and merchants can generate it
3. hover text resolves before and during combat
4. the target prompt appears when more than one player is valid
5. the chosen player draws the cards
6. the base texture and optional outline render in every potion slot
7. a save containing the potion reloads

If `PotionModel.Pool` throws, registration is missing or late. If the model works but the bottle is blank, inspect the fixed atlas-resource path inside the final PCK.
