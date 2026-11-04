# 15. State, Saves, Cloning, And Randomness

A model becomes considerably more demanding once it remembers something. A relic that counts played cards must distinguish its canonical definition from its owned instance, preserve progress across saves, update its counter, and avoid counting another player's actions. Random effects introduce another requirement: the same run state must lead to the same decisions after a reload.

This chapter follows those boundaries in `v0.107.1`. The examples extend the [custom relic tutorial](12-custom-relics.md) and the [act event tutorial](13-custom-act-events.md).

## Choose The Lifetime First

Before choosing a field, decide what its value belongs to:

| State | Appropriate lifetime |
| --- | --- |
| Base amount or threshold | Canonical definition and dynamic variables |
| Relic progress across rooms | Mutable relic instance with saved properties |
| A temporary target during resolution | Mutable instance, cleared at the relevant lifecycle boundary |
| Panel selection or animation progress | UI node, recreated with the panel |
| A sequence of random decisions | Seeded gameplay RNG with an explicitly owned lifetime |

A static field belongs to the process. It is shared across copies and players, survives scene replacement, and is not automatically restored from a run save. It is therefore a poor default for gameplay progress.

## A Relic That Remembers Played Cards

The following relic draws one card after every fifth owner play reported to `AfterCardPlayedLate` while combat is still in progress. Its progress carries between combats and is persisted in the run save. A play that ends combat may not reach this hook and therefore may not advance the counter.

```csharp
using System.Threading.Tasks;
using MegaCrit.Sts2.Core.Commands;
using MegaCrit.Sts2.Core.Entities.Cards;
using MegaCrit.Sts2.Core.Entities.Relics;
using MegaCrit.Sts2.Core.GameActions.Multiplayer;
using MegaCrit.Sts2.Core.Models;
using MegaCrit.Sts2.Core.Saves.Runs;

namespace FieldNotes.Relics;

public sealed class ResearchLedgerRelic : RelicModel
{
    private const int Threshold = 5;
    private int _cardsRecorded;

    public override RelicRarity Rarity => RelicRarity.Uncommon;
    public override bool ShowCounter => true;
    public override int DisplayAmount => CardsRecorded;

    public override string PackedIconPath =>
        "res://FieldNotes/images/relics/research_ledger.png";

    protected override string PackedIconOutlinePath =>
        "res://FieldNotes/images/relics/research_ledger_outline.png";

    protected override string BigIconPath =>
        "res://FieldNotes/images/relics/research_ledger_large.png";

    [SavedProperty]
    public int CardsRecorded
    {
        get => _cardsRecorded;
        set
        {
            AssertMutable();
            _cardsRecorded = ((value % Threshold) + Threshold) % Threshold;
            InvokeDisplayAmountChanged();
        }
    }

    public override async Task AfterCardPlayedLate(
        PlayerChoiceContext choiceContext,
        CardPlay cardPlay)
    {
        if (cardPlay.Card.Owner != Owner)
        {
            return;
        }

        CardsRecorded++;
        if (CardsRecorded != 0)
        {
            return;
        }

        Flash();
        await CardPileCmd.Draw(choiceContext, 1m, Owner);
    }
}
```

Register `ResearchLedgerRelic` in `SharedRelicPool` during the initializer, using the same [registration pattern](12-custom-relics.md#registration) as the earlier relic. Supply all three images and add these entries to the mod's `relics.json`:

```json
{
  "RESEARCH_LEDGER_RELIC.title": "Research Ledger",
  "RESEARCH_LEDGER_RELIC.description": "After every 5 card plays you complete while combat remains in progress, draw 1 card. Progress carries between combats.",
  "RESEARCH_LEDGER_RELIC.flavor": "The fifth observation always raises another question."
}
```

The owner check matters because hook dispatch can present actions performed by other players. The example counts every play that reaches this hook for the owner, including applicable automatic plays and replays; it does not claim to count only cards played from the hand. If that distinction matters to the design, inspect the current `CardPlay` construction and the relevant vanilla effect before adding a filter.

The counter resets before drawing. Reactions triggered by the draw therefore observe the next cycle's state. Keeping the draw in the awaited command chain preserves the game's choice context and sequencing. At this point the played card has not necessarily left the Play pile; the late hook is not a notification that final pile movement has completed.

This example intentionally delays its draw until after ordinary post-play listeners. Use ordinary `AfterCardPlayed` if that ordering is not part of the desired effect.

## What SavedProperty Actually Saves

`SerializableRelic` stores a model ID plus an optional `SavedProperties` payload. That payload reads attributed properties through reflection and records their property names. The backing field is not the serialization contract; `CardsRecorded` is.

The current writer explicitly handles these value families:

- `int`, `bool`, and `string`
- enums, encoded as integers
- `int[]` and enum arrays
- `ModelId`
- `SerializableCard`
- `List<SerializableCard>`

Do not infer arbitrary-object support from the JSON-based save format. A dictionary, `List<int>`, Godot node, or custom class is not automatically supported by this attribute. Unsupported non-null values cause a serialization exception. Null values are warned about rather than stored as ordinary supported values.

Use simple saved values and reconstruct derived state from them. Keep scene nodes, textures, event delegates, and animation tasks out of the save representation. The fact that a property saves also does not make arbitrary mutations into multiplayer commands: state still needs to change through deterministic gameplay flow.

## Save Compatibility Is An Interface

Two names become persistent interfaces:

1. the model entry derived from the class name
2. each attributed property name

Renaming `ResearchLedgerRelic` or `CardsRecorded` can invalidate or lose old state unless explicit migration handles it. A renamed property is not automatically mapped to its replacement.

Changing a threshold is also a gameplay migration. The example normalizes a loaded value into the new counter range, but that policy changes the player's progress. Choose and test a deliberate policy rather than treating a successful deserialize as proof of compatibility.

Test a run saved with progress `4`, reload it in a fresh process, then play one card. Test the same sequence across a combat boundary. The draw should occur exactly once, and the displayed count should match the restored state.

## Cloning Is A Separate Operation

Serialization and cloning solve different problems. `AbstractModel.MutableClone` begins with `MemberwiseClone`, sets mutability, calls `DeepCloneFields`, then calls `AfterCloned`. A shallow copy duplicates a reference to a collection or delegate; it does not duplicate the referenced object.

For a custom mutable collection, the pattern is:

```csharp
protected override void DeepCloneFields()
{
    base.DeepCloneFields();
    _visitedEntries = new HashSet<string>(_visitedEntries);
}
```

This is a fragment for a model that declares `_visitedEntries` as `HashSet<string>`. It protects clones from sharing the same set. It does not make the set a supported saved property.

For events declared by the custom model, clear the copied delegates:

```csharp
protected override void AfterCloned()
{
    base.AfterCloned();
    ProgressChanged = null;
}
```

Here `ProgressChanged` is an event declared by that model. Clearing its copied delegate prevents a clone from notifying subscribers attached to the source instance and retaining UI objects through those subscribers.

Prefer the model-specific `ToMutable()` factory when obtaining a gameplay instance from a canonical definition. `ClonePreservingMutability()` returns the canonical instance unchanged, or clones an already mutable instance. These semantics matter in preview and inspect code: do not assume every operation called "clone" produces an independent mutable model.

## Randomness Has State Too

The game's `Rng` exposes a seed and a counter. Reproducing a sequence requires both the same initial state and the same calls in the same order. A tooltip that rolls an outcome on hover can therefore change gameplay even when nothing visible has been selected.

`Rng.Chaotic` is explicitly non-deterministic across saves. Reserve it for effects that do not influence gameplay. Avoid `System.Random`, clock-based seeds, or Godot random functions for rewards, damage, event options, or target selection that must be reproducible.

### Event Rolls

`EventModel` provides a per-event `Rng` after the event begins and receives its owner. It is unavailable on the canonical event. Roll dynamic event values in `CalculateVars`, then use the resulting value for both text and the selected effect.

For the existing observatory event, replace its fixed gold calculation with this override:

```csharp
public override void CalculateVars()
{
    DynamicVars.Gold.BaseValue = Rng.NextInt(40, 81);
}
```

The upper bound is exclusive: this produces 40 through 80. Do not roll again in the option's getter or click handler; otherwise the displayed offer and the actual reward can disagree.

### Content-Specific Streams

The current API also offers:

```text
Rng(Player player, ModelId id, uint mixin = 0, int counter = 0)
```

Its seed derives from the run seed, player slot, model entry, and optional mixin. Player slot is used rather than network ID so a changed network identity need not change a daily-run outcome.

Creating this RNG again with identical arguments starts the same sequence. That can be useful for a stable offer, but it does not provide a fresh roll for each occurrence. For repeated occurrences, own the stream explicitly or use a deterministic occurrence mixin and preserve the state needed to reconstruct it. Do not call runtime `GetHashCode()` to manufacture a stable mixin.

Candidate order is part of the random algorithm. When selecting from a set of models, construct the candidate list in a deterministic order, such as ordinal model-entry order, before drawing an index. The same seed cannot compensate for different lists or different list ordering on peers.

## Useful Failure Signatures

| Symptom | Likely boundary to inspect |
| --- | --- |
| Progress disappears after restart | Missing saved property or changed property name |
| Two instances share progress | Static field or shallow-copied mutable reference |
| Preview triggers the original UI | Copied event delegate |
| Save fails only after obtaining an item | Unsupported saved-property value type |
| Offer changes when hovered | RNG consumed in a presentation path |
| Same item always rolls the same result | RNG recreated with identical seed arguments |
| Multiplayer counts other players' cards | Missing ownership filter in the hook |

## Verification

The full relic example is intended for the same assembly and resources as the earlier content examples. Check compilation separately from runtime behavior:

1. grant two mutable instances in an isolated test and confirm their counters are independent
2. play four cards, save, restart, and confirm the fifth triggers one draw
3. cross a combat boundary with partial progress
4. inspect and hover the relic repeatedly; neither operation should change progress or gameplay RNG
5. test another player's card plays in multiplayer
6. test a save from the previous mod release after changing a saved field or threshold

Use [save troubleshooting](08-debugging-and-fast-iteration.md#save-state-troubleshooting) for file locations and [hook selection](05-entrypoints-harmony-and-hooks.md#semantic-hooks) for resolution timing. Compilation verifies API use; these scenarios verify the intended behavior.
