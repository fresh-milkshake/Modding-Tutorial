# 12. Custom Relics

Relics are persistent run models that receive semantic hooks while owned by a player. The example adds `SurveyorLensRelic`, a common relic that grants six Block at the start of combat.

## Implementation

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using MegaCrit.Sts2.Core.Commands;
using MegaCrit.Sts2.Core.Entities.Relics;
using MegaCrit.Sts2.Core.HoverTips;
using MegaCrit.Sts2.Core.Localization.DynamicVars;
using MegaCrit.Sts2.Core.Models;
using MegaCrit.Sts2.Core.ValueProps;

namespace FieldNotes.Relics;

public sealed class SurveyorLensRelic : RelicModel
{
    public override RelicRarity Rarity => RelicRarity.Common;

    public override string PackedIconPath =>
        "res://FieldNotes/images/relics/surveyor_lens.png";

    protected override string PackedIconOutlinePath =>
        "res://FieldNotes/images/relics/surveyor_lens_outline.png";

    protected override string BigIconPath =>
        "res://FieldNotes/images/relics/surveyor_lens_large.png";

    protected override IEnumerable<DynamicVar> CanonicalVars =>
    [
        new BlockVar(6m, ValueProp.Unpowered)
    ];

    protected override IEnumerable<IHoverTip> ExtraHoverTips =>
    [
        HoverTipFactory.Static(StaticHoverTip.Block)
    ];

    public override async Task BeforeCombatStart()
    {
        Flash();

        await CreatureCmd.GainBlock(
            Owner.Creature,
            DynamicVars.Block,
            cardPlay: null);
    }
}
```

`RelicModel` already participates in combat hooks while it is present in player state. Overriding `BeforeCombatStart` is therefore sufficient; no Harmony patch is required.

`ValueProp.Unpowered` matches an effect that should not be modified as a card move. Choose value properties by inspecting a vanilla effect with the same semantics, not by reusing the first enum value that compiles.

## Registration

```csharp
using FieldNotes.Relics;
using MegaCrit.Sts2.Core.Modding;
using MegaCrit.Sts2.Core.Models.RelicPools;

ModHelper.AddModelToPool<SharedRelicPool, SurveyorLensRelic>();
```

Pool membership is mandatory. `RelicModel.Pool` uses a matching search rather than a nullable lookup, and several hover, inspect, color, and collection paths access it.

## Localization

Create `godot/FieldNotes/localization/eng/relics.json`:

```json
{
  "SURVEYOR_LENS_RELIC.title": "Surveyor Lens",
  "SURVEYOR_LENS_RELIC.description": "At the start of combat, gain {Block:diff()} [gold]Block[/gold].",
  "SURVEYOR_LENS_RELIC.flavor": "Every path looks shorter after it has been measured.",
  "SURVEYOR_LENS_RELIC.selectionScreenPrompt": "Choose Surveyor Lens."
}
```

`Title`, `DynamicDescription`, and `Flavor` derive their keys from the model entry. Event integrations can also use `eventDescription`; if it is absent, the base model falls back to the ordinary description.

## Icon Contract

Relics appear in several visual contexts:

- packed small icon in the top bar and lists
- packed outline used by some hover and presentation states
- large image in inspect and reward views

The base class normally derives these paths from the lower-case model ID. The example overrides all three paths to keep resources namespaced:

```text
res://FieldNotes/images/relics/surveyor_lens.png
res://FieldNotes/images/relics/surveyor_lens_outline.png
res://FieldNotes/images/relics/surveyor_lens_large.png
```

Do not validate only the small icon. A relic can appear correct in the top bar and fail when a reward, event option, or inspect view requests the large art.

## Canonical And Mutable Instances

`ModelDb.Relic<SurveyorLensRelic>()` is canonical. To grant the relic:

```csharp
var relic = ModelDb.Relic<SurveyorLensRelic>().ToMutable();
await RelicCmd.Obtain(relic, player);
```

`RelicCmd.Obtain` establishes ownership and runs obtain behavior. Directly inserting a canonical relic into a collection bypasses lifecycle and mutability checks.

Use `AfterObtained` and `AfterRemoved` for state changes tied to ownership. Combat hooks should assume `Owner` exists because they are reached through the owned relic collection.

## Stateful Relics

For counters and persistent fields, inspect:

- `ShowCounter`
- `DisplayAmount`
- saved-property support used by vanilla relics
- `InvokeDisplayAmountChanged`
- `AfterObtained` and `AfterRemoved`

Mutate only mutable instances. If a custom field must survive save/load, use the same save-property mechanism as a current stateful vanilla relic and test a full process restart.

## Unlock And Collection Visibility

Registration makes the relic part of `SharedRelicPool`. The current shared-pool unlock filter removes specific vanilla epoch lists, so the custom relic remains in the unlocked set unless the mod adds its own gate.

The relic collection separately tracks discovered relic IDs. An undiscovered but unlocked relic can appear in a locked presentation. Verify both natural generation and collection display before adding Harmony patches to unlock code.

## Validation Matrix

Verify:

1. the shared relic pool contains the model
2. a reward can generate it
3. `RelicCmd.Obtain` creates an owned mutable instance
4. the start-of-combat hook fires once per combat
5. Block uses the expected modifier semantics
6. small, outline, and large art render
7. hover and inspect views do not access a missing pool
8. a run containing the relic saves and reloads

The most diagnostic failure is `RelicModel.get_Pool()` throwing a missing-element exception. Fix registration before investigating UI code.
