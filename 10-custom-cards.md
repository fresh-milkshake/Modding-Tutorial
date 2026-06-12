# 10. Custom Cards

A custom card is the most direct example of the STS2 content lifecycle. The class defines rules, `ModelDb` gives it an identity, a card pool makes it selectable, localization supplies text, and a portrait resource completes presentation.

This chapter adds `FieldNotesCard`, a colorless skill that grants Block and draws a card. Its upgrade increases Block.

## Model Identity

`ModelDb` derives the entry ID from the class name:

```text
FieldNotesCard -> FIELD_NOTES_CARD
```

That entry controls localization and default resource slugs. Renaming the class after release changes the model ID and can break saves that contain the old card.

## Implementation

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using MegaCrit.Sts2.Core.Commands;
using MegaCrit.Sts2.Core.Entities.Cards;
using MegaCrit.Sts2.Core.GameActions.Multiplayer;
using MegaCrit.Sts2.Core.Localization.DynamicVars;
using MegaCrit.Sts2.Core.Models;
using MegaCrit.Sts2.Core.ValueProps;

namespace FieldNotes.Cards;

public sealed class FieldNotesCard : CardModel
{
    public override bool GainsBlock => true;

    public override string PortraitPath =>
        "res://FieldNotes/images/cards/field_notes.png";

    protected override IEnumerable<DynamicVar> CanonicalVars =>
    [
        new BlockVar(6m, ValueProp.Move),
        new CardsVar(1)
    ];

    public FieldNotesCard()
        : base(
            canonicalEnergyCost: 1,
            type: CardType.Skill,
            rarity: CardRarity.Uncommon,
            targetType: TargetType.Self)
    {
    }

    protected override async Task OnPlay(
        PlayerChoiceContext choiceContext,
        CardPlay cardPlay)
    {
        await CreatureCmd.GainBlock(
            Owner.Creature,
            DynamicVars.Block,
            cardPlay);

        await CardPileCmd.Draw(
            choiceContext,
            DynamicVars.Cards.BaseValue,
            Owner);
    }

    protected override void OnUpgrade()
    {
        DynamicVars.Block.UpgradeValueBy(3m);
    }
}
```

The constructor defines four independent properties:

- canonical energy cost
- card type
- rarity
- target type

`OnPlay` uses game commands rather than direct state mutation. The commands preserve command ordering, visuals, and multiplayer choice context. `GainsBlock` is presentation metadata used by systems that classify the card's behavior.

## Dynamic Variables

`CanonicalVars` defines the canonical values copied into each mutable card instance. `DynamicVars` exposes the instance values after upgrades and runtime modifiers.

Use the typed value required by the command:

- `CreatureCmd.GainBlock` accepts the block dynamic variable
- `CardPileCmd.Draw` accepts the card count

The upgrade changes the mutable `BlockVar`; localization can then display the base and upgraded values with `{Block:diff()}`.

## Registration

Register the card before any pool is enumerated:

```csharp
using FieldNotes.Cards;
using MegaCrit.Sts2.Core.Modding;
using MegaCrit.Sts2.Core.Models.CardPools;

ModHelper.AddModelToPool<ColorlessCardPool, FieldNotesCard>();
```

`ColorlessCardPool` determines the visual frame, energy color, reward family, and library grouping. Registering in a character pool instead makes the card part of that character's ordinary card ecosystem.

Do not register the same model in several ordinary pools without checking every consumer of `CardModel.Pool`. The property selects the first pool containing the card ID, so ambiguous membership can produce presentation and filtering behavior that depends on pool order.

## Localization

Create `godot/FieldNotes/localization/eng/cards.json`:

```json
{
  "FIELD_NOTES_CARD.title": "Field Notes",
  "FIELD_NOTES_CARD.description": "Gain {Block:diff()} [gold]Block[/gold].\nDraw {Cards:diff()} card."
}
```

Pack it as:

```text
res://FieldNotes/localization/eng/cards.json
```

Add equivalent files for every supported language. Missing non-English files normally fall back according to the game's localization behavior, but a released mod should not claim a language it has not tested.

## Portrait

Because `PortraitPath` is virtual, the example uses a namespaced PNG:

```text
res://FieldNotes/images/cards/field_notes.png
```

The card frame, banner, energy icon, and other chrome come from the selected pool. Only the portrait is custom.

The default portrait path uses an atlas resource below `res://images/atlases/card_atlas.sprites/<pool>/`. Overriding the path avoids adding a bridge resource to that shared native directory. Verify the PNG imports as a `Texture2D` and appears in both combat and the card library.

## Creating A Mutable Card

`ModelDb.Card<FieldNotesCard>()` returns the canonical definition. Gameplay requires a mutable, owner-bound instance:

```csharp
var card = player.RunState.CreateCard<FieldNotesCard>(player);
await CardPileCmd.Add(card, PileType.Deck);
```

Use the run state's factory when adding a card to a deck. It establishes ownership and other runtime fields expected by commands and saves.

## Upgrade Design

The base class calls `OnUpgrade` while applying an upgrade level. More complex cards can also override:

- `MaxUpgradeLevel`
- `GetResultPileType`
- canonical keywords and tags
- extra description arguments
- creation and transformation callbacks

Keep upgrades deterministic and derived from canonical variables. UI previews clone and transform cards; code that assumes every preview is the original deck instance can apply an upgrade twice.

## Validation Matrix

Verify:

1. `ModelDb.Card<FieldNotesCard>()` resolves
2. `ColorlessCardPool.AllCards` contains the model
3. a reward or direct grant creates a mutable owner-bound card
4. base and upgraded descriptions show correct values
5. the portrait renders in combat, deck view, reward view, and card library
6. the card moves to the normal result pile after play
7. a run containing the card saves and reloads

If the card exists but never appears in rewards, the problem is selection or unlock filtering. If the effect runs but card movement breaks, the problem is the play lifecycle rather than registration.
