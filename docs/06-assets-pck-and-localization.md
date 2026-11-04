# 06. Content Infrastructure, Assets, And Localization

Custom content crosses four independent systems: type discovery, selection pools, presentation resources, and visibility rules. Treating registration as a single operation produces models that work in one screen and fail in another.

## Model Discovery

During initialization, `ModelDb` combines vanilla `AbstractModel` subtypes with concrete subtypes found in loaded mod assemblies. It creates one canonical instance of every public parameterless model type and assigns an ID derived from the base model category and class name.

For example, a class named `FieldNotesCard : CardModel` receives the entry slug `FIELD_NOTES_CARD`. The base card class then requests:

```text
cards: FIELD_NOTES_CARD.title
cards: FIELD_NOTES_CARD.description
```

Discovery makes a model addressable through `ModelDb`; it does not make the model selectable by rewards, shops, events, or libraries.

## Pool Membership

Cards, potions, and relics use `IPoolModel` implementations. Register them during `[ModInitializer]`:

```csharp
ModHelper.AddModelToPool<ColorlessCardPool, FieldNotesCard>();
ModHelper.AddModelToPool<SharedPotionPool, BottledInsightPotion>();
ModHelper.AddModelToPool<SharedRelicPool, SurveyorLensRelic>();
```

The pool caches its generated content and then freezes mod additions. Late registration throws an exception. Register first; only then patch or access systems that may force `ModelDb.Preload` or enumerate a pool.

Pool membership affects more than random generation. `PotionModel.Pool` and `RelicModel.Pool` search all current pools and throw when no matching membership exists. Hover, inspect, color, and catalog paths can therefore fail even if the model was granted directly by debug code.

## Events Are Different

Current act events do not use `ModHelper.AddModelToPool`. `ActModel.GenerateRooms` combines the concrete act's `AllEvents` with `ModelDb.AllSharedEvents`. A custom event type is discovered automatically, but it must be appended to one of those lists through a narrow Harmony patch. Chapter 13 shows an act-specific patch.

## Unlock And Catalog Visibility

Ask separate questions:

1. Can gameplay select the model?
2. Does the current `UnlockState` include it?
3. Has the profile discovered it?
4. Does the relevant library choose to display undiscovered content?
5. Does the model itself allow library display?

The current shared card, potion, and relic pool filters remove known vanilla epoch entries. A newly registered model is therefore generally left in the unlocked result unless a mod adds its own gate. Discovery UI can still display it as unseen until the profile records it.

For cards, the `CardModel` constructor also has `shouldShowInCardLibrary`, which defaults to `true`. Generation and library visibility are not the same switch.

## Localization Contract

For every loaded mod, `ModManager.GetModdedLocTables(language, file)` checks:

```text
res://<manifest id>/localization/<language>/<file>
```

For the example manifest ID `FieldNotes`:

```text
res://FieldNotes/localization/eng/cards.json
res://FieldNotes/localization/eng/potions.json
res://FieldNotes/localization/eng/relics.json
res://FieldNotes/localization/eng/events.json
```

The files are flat JSON key/value maps:

```json
{
  "FIELD_NOTES_CARD.title": "Field Notes",
  "FIELD_NOTES_CARD.description": "Gain {Block:diff()} [gold]Block[/gold].\nDraw {Cards:diff()} card."
}
```

Use vanilla table names and language codes. A syntactically valid file under `res://localization/...` will not be merged as mod localization because the loader includes the manifest ID in its lookup path.

## Dynamic Variables

Descriptions can interpolate `DynamicVar` values defined by a model:

- `{Block:diff()}` displays base and upgraded block
- `{Damage:diff()}` displays damage
- `{Cards:diff()}` displays a card count
- `{Energy:energyIcons()}` displays energy icons

The variable name and formatting function must match the actual variable set. Copy syntax from a vanilla model using the same `DynamicVar` type.

## Asset Paths

Base models resolve resources differently:

- cards expose virtual `PortraitPath`
- relics expose virtual packed, outline, and large icon paths
- potions use fixed atlas-resource paths derived from the model ID
- default events use a fixed portrait path derived from the event ID
- custom event layouts resolve a scene from the event slug

This distinction determines whether to override a property or place a resource at a native path. Chapters 10-13 document each contract.

Namespaced resource paths reduce collision risk:

```text
res://FieldNotes/images/cards/field_notes.png
res://FieldNotes/images/relics/surveyor_lens.png
```

Fixed native lookup paths may still require small `.tres` bridge resources under `res://images/...`. Use unique model slugs to avoid collisions with vanilla or other mods.

## Building The PCK

A content PCK may contain:

- namespaced localization tables
- PNG files and generated `.import` data
- `AtlasTexture` resources
- `.tscn` scenes
- themes, materials, shaders, and audio resources

The PCK does not need a second mod manifest. The external `<id>.json` controls loading.

After packing, list the final archive and verify the exact internal paths. A successful pack command proves only that a file was produced.

For textures, retain the source-path `.import` remap and the generated `.godot/imported/...` resource that it references. A `.tres` pointing at a PNG does not itself generate texture imports. Export through a matching Godot project or pack the already imported resources together; do not assume copying raw PNGs into a PCK is sufficient for exported-game resource loading.

## Resource Validation

Test each model in at least these contexts:

- random generation or reward selection
- direct creation through `ModelDb`
- hover and inspect view
- compendium or library view
- save and reload after obtaining the item

This matrix catches missing pool membership, bad art paths, hidden unlock state, and serialization failures earlier than ordinary playtesting.

## Related Sections

For complete implementations, continue with [cards](10-custom-cards.md), [potions](11-custom-potions.md), [relics](12-custom-relics.md), or [act events](13-custom-act-events.md). Their [validation matrices](08-debugging-and-fast-iteration.md#update-regression-matrix) complement registration checks.
