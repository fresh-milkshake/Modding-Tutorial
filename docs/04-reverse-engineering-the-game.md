# 04. Reverse-Engineering The Game

STS2 does not currently expose a separate, versioned mod SDK. Reverse engineering is therefore ordinary compatibility work: inspect the release that will execute the mod, identify the narrowest supported integration point, and record the evidence behind the decision.

## Evidence Sources

Use each source for the question it can answer:

| Source | Best for |
| --- | --- |
| `release_info.json` | exact release and commit |
| `sts2.runtimeconfig.json` | target framework and bundled runtime |
| `sts2.dll` | types, signatures, call order, hooks, and loader behavior |
| `SlayTheSpire2.pck` | scenes, localization, images, and exact `res://` paths |
| runtime logs | what the executable actually attempted |
| clean installation test | whether all layers work together |

A decompiler can produce awkward names for compiler-generated arrays and state machines. Treat those artifacts as presentation noise; focus on public signatures, control flow, path construction, and method calls.

## Start From The Domain Object

Search by the object the player interacts with, not by a guessed patch site:

- cards: `CardModel`, a simple vanilla card, its pool, and `CardPileCmd`
- potions: `PotionModel`, a simple potion, its pool, and use commands
- relics: `RelicModel`, a simple hook-based relic, and its pool
- events: `EventModel`, `EventOption`, the target `ActModel`, and `EventRoom`
- UI: the concrete native screen, its stack, host, tooltip layer, and focus context

The nearest simple vanilla implementation is usually the best executable specification. For example, `Backflip`, `SwiftPotion`, `Anchor`, and `ThisOrThat` demonstrate the current constructor, dynamic-variable, async command, and event-option patterns without large amounts of unrelated logic.

## Assembly Workflow

With ILSpy command-line tooling, a focused query has the form:

```powershell
ilspycmd --disable-updatecheck `
  -t 'MegaCrit.Sts2.Core.Models.Cards.Backflip' `
  '<STS2>\data_sts2_windows_x86_64\sts2.dll'
```

Repeat the query for the base model and the pool. Do not copy a concrete implementation before checking which members are inherited and which paths are computed by the base class.

High-value current types include:

- `MegaCrit.Sts2.Core.Modding.ModManager`
- `MegaCrit.Sts2.Core.Modding.ModManifest`
- `MegaCrit.Sts2.Core.Modding.ModHelper`
- `MegaCrit.Sts2.Core.Helpers.ReflectionHelper`
- `MegaCrit.Sts2.Core.Models.ModelDb`
- `MegaCrit.Sts2.Core.Hooks.Hook`

The current loader's directory scan and initializer methods are private implementation details. Inspect them when validating behavior, but do not build mod code that calls them directly.

## PCK Workflow

A compatible PCK tool such as the locally available `godotpcktool.exe` can list or extract selected files:

```powershell
godotpcktool.exe `
  --pack '<STS2>\SlayTheSpire2.pck' `
  --action list `
  --include-regex-filter '^localization/eng/(cards|potions|relics|events)\.json$'
```

Prefer selective extraction. The main pack is large, while most questions concern one table, scene, or image family.

Use the PCK to verify:

- whether localization is a flat key/value table or a nested object
- whether a model expects a PNG, imported texture, atlas resource, or scene
- the exact slug and case used in paths
- the native scene hierarchy before reusing a UI screen

## Trace The Full Contract

For new content, answer these questions before implementation:

1. How does `ModelDb` discover the type?
2. Which pool or list allows generation to select it?
3. Which localization table and keys does the base class construct?
4. Which resource paths are fixed, virtual, or derived from the pool?
5. Which unlock or catalog filters can hide it?
6. Which mutable owner state is required when the effect runs?
7. Which save representation stores its model ID or fields?

This prevents partial integrations. A type can exist in `ModelDb` but never be generated; it can be generated but fail in hover UI; it can render in combat but disappear from the library; it can work until a save is loaded.

## Update Maintenance

After every STS2 update:

1. compare `release_info.json` and the assembly hash
2. rebuild against the new shipped DLLs
3. re-decompile every patched method and every inherited base model used by the mod
4. re-list PCK paths used by custom resources
5. test a clean new run and a save created by the previous release

Compilation is only the first compatibility check. Harmony targets, reflection names, resource paths, and lifecycle order can change without producing compile errors.

## Shipped XML Documentation

The current data directory includes `sts2.xml` beside `sts2.dll`. Keep the XML alongside the referenced assembly so an IDE can display its summaries and parameter documentation. Search its member entries before decompiling large types; inspect the implementation when lifecycle or ordering remains unclear.

XML comments describe intent, while the installed assembly defines the actual signature. Use [the API inventory](appendix-api-inventory.md) to locate relevant types, then validate the exact member against both sources.
