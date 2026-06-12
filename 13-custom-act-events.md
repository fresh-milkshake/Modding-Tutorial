# 13. Custom Act Events

Map events use a different registration path from cards, potions, and relics. `ModelDb` discovers an `EventModel` subtype, but an act selects events only from its `AllEvents` list plus `ModelDb.AllSharedEvents`. The example adds `AbandonedObservatoryEvent` to the first act.

The event offers two choices: take the example card or sell the collected notes for gold.

## Event Model

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using FieldNotes.Cards;
using MegaCrit.Sts2.Core.Commands;
using MegaCrit.Sts2.Core.Entities.Cards;
using MegaCrit.Sts2.Core.Events;
using MegaCrit.Sts2.Core.HoverTips;
using MegaCrit.Sts2.Core.Localization.DynamicVars;
using MegaCrit.Sts2.Core.Models;

namespace FieldNotes.Events;

public sealed class AbandonedObservatoryEvent : EventModel
{
    protected override IEnumerable<DynamicVar> CanonicalVars =>
    [
        new GoldVar(60),
        new StringVar("Card", ModelDb.Card<FieldNotesCard>().Title)
    ];

    protected override IReadOnlyList<EventOption> GenerateInitialOptions()
    {
        return
        [
            new EventOption(
                this,
                TakeNotes,
                InitialOptionKey("TAKE_NOTES"),
                HoverTipFactory.FromCardWithCardHoverTips<FieldNotesCard>()),

            new EventOption(
                this,
                SellNotes,
                InitialOptionKey("SELL_NOTES"))
        ];
    }

    private async Task TakeNotes()
    {
        var owner = Owner
            ?? throw new InvalidOperationException("Event has no owner.");

        var card = owner.RunState.CreateCard<FieldNotesCard>(owner);
        var addedCard = await CardPileCmd.Add(card, PileType.Deck);
        CardCmd.PreviewCardPileAdd(addedCard, 2f);

        SetEventFinished(L10NLookup(
            "ABANDONED_OBSERVATORY_EVENT.pages.TAKE_NOTES.description"));
    }

    private async Task SellNotes()
    {
        var owner = Owner
            ?? throw new InvalidOperationException("Event has no owner.");

        await PlayerCmd.GainGold(DynamicVars.Gold.IntValue, owner);

        SetEventFinished(L10NLookup(
            "ABANDONED_OBSERVATORY_EVENT.pages.SELL_NOTES.description"));
    }
}
```

`GenerateInitialOptions` returns the first page's choices. An option receives an async delegate, a localization key, and optional hover tips. Passing `null` instead of a delegate creates a locked option.

The card choice creates a mutable card through the run state, adds it through `CardPileCmd`, and shows the same preview flow used by vanilla events. The gold choice uses `PlayerCmd` rather than mutating the player's currency directly.

## Localization

Create `godot/FieldNotes/localization/eng/events.json`:

```json
{
  "ABANDONED_OBSERVATORY_EVENT.title": "Abandoned Observatory",
  "ABANDONED_OBSERVATORY_EVENT.pages.INITIAL.description": "A cracked telescope still points toward a motionless star. Notes cover the floor, each describing a path through the Spire that no map records.",
  "ABANDONED_OBSERVATORY_EVENT.pages.INITIAL.options.TAKE_NOTES.title": "Take Notes",
  "ABANDONED_OBSERVATORY_EVENT.pages.INITIAL.options.TAKE_NOTES.description": "Add [gold]{Card}[/gold] to your Deck.",
  "ABANDONED_OBSERVATORY_EVENT.pages.INITIAL.options.SELL_NOTES.title": "Sell Notes",
  "ABANDONED_OBSERVATORY_EVENT.pages.INITIAL.options.SELL_NOTES.description": "Gain [gold]{Gold}[/gold] Gold.",
  "ABANDONED_OBSERVATORY_EVENT.pages.TAKE_NOTES.description": "You copy the most coherent route and leave before the observatory settles further into the stone.",
  "ABANDONED_OBSERVATORY_EVENT.pages.SELL_NOTES.description": "A passing courier pays well for maps that might be true."
}
```

`InitialOptionKey("TAKE_NOTES")` expands to the class-derived event prefix and the `pages.INITIAL.options` path. Keeping this convention lets event history and generic event UI resolve the same keys as vanilla events.

## Portrait

The default event layout derives a fixed portrait path from the model ID:

```text
res://images/events/abandoned_observatory_event.png
```

`CreateInitialPortrait` is not virtual in the current base class, so the simplest default-layout event supplies a texture at that exact path. The slug is unique, which limits collision risk.

Pack:

```text
godot/images/events/abandoned_observatory_event.png
```

If the event needs a completely different composition, override `LayoutType` with `EventLayoutType.Custom` and provide the scene expected by `EventModel`. The root control must implement `ICustomEventNode` and initialize itself from the event model. That is a larger UI feature and should be built only after the default event flow works.

## Add The Event To An Act

There is no current `ModHelper` event pool. Patch the concrete act's getter:

```csharp
using System.Collections.Generic;
using System.Linq;
using FieldNotes.Events;
using HarmonyLib;
using MegaCrit.Sts2.Core.Models;
using MegaCrit.Sts2.Core.Models.Acts;

namespace FieldNotes.Patches;

[HarmonyPatch(
    typeof(Overgrowth),
    nameof(Overgrowth.AllEvents),
    MethodType.Getter)]
internal static class OvergrowthEventsPatch
{
    private static void Postfix(ref IEnumerable<EventModel> __result)
    {
        var customEvent = ModelDb.Event<AbandonedObservatoryEvent>();

        if (__result.All(e => e.Id != customEvent.Id))
        {
            __result = __result.Append(customEvent);
        }
    }
}
```

This patch is narrow: it changes only the event list of `Overgrowth`. Add equivalent patches for other acts only when the event belongs there.

For an event intended to appear in every act, patch the getter for `ModelDb.AllSharedEvents` before the model database is preloaded. Keep duplicate protection because aggregate lists can be evaluated more than once.

## Eligibility

Override `IsAllowed(IRunState runState)` for conditions such as:

- a required relic or card is present
- the event has not already been visited
- the run is before or after a specific act milestone
- the event is incompatible with multiplayer
- another modded state is available

Eligibility is evaluated during room-set validation and event selection. Keep it deterministic for a given run state; random decisions belong in the event's seeded `Rng` and `CalculateVars`.

## Multi-Page Events

An option delegate can call `SetEventState` instead of `SetEventFinished` to present another description and another option list. Use this for negotiations, repeated costs, or branching outcomes.

Related capabilities include:

- locked options with a null delegate
- `ThatDoesDamage` and `ThatDecreasesMaxHp` death warnings
- relic options with native relic presentation
- event combat and resume flow
- custom background scenes and VFX
- shared multiplayer events

Inspect a vanilla event that uses the same capability. Event combat, shared state, and custom layouts have additional ownership rules that a simple option event does not exercise.

## Validation Matrix

Verify:

1. `ModelDb.Event<AbandonedObservatoryEvent>()` resolves
2. `Overgrowth.AllEvents` contains the model after patches apply
3. a newly generated act can place the event in an unknown room
4. both options render text and hover content
5. each option mutates state exactly once
6. the event completes and returns to normal room flow
7. event history records the chosen option correctly
8. a save made before entering and during the event reloads safely

Existing acts generate their room sets when a run begins. Adding the patch to a DLL does not retroactively insert the event into an already generated act save.
