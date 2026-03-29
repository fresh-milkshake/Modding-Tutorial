# 08. Debugging And Fast Iteration

## Start With Native Signals

Before adding custom tooling, check:

- `sts2_stdout.log`
- `sts2_stderr.log`
- the mod loader UI
- in-game behavior

Also check Godot-side logs under `%APPDATA%\SlayTheSpire2\logs\...`. They can contain managed traces that never appear in `sts2_stdout.log`.

These sources matter because they answer different questions. The mod loader UI tells you whether the game recognized the payload at all. The runtime logs tell you what the game attempted to load and which failures surfaced during startup or execution. In-game behavior tells you whether the intended integration point actually changed gameplay rather than merely loading without effect.

## Save Paths Worth Knowing

For run-state debugging on this machine, active saves can exist in both:

- `%APPDATA%\SlayTheSpire2\steam\<SteamId>\...`
- `C:\Program Files (x86)\Steam\userdata\<SteamAccountId>\2868840\remote\...`

That includes modded runs under `modded\profile1\saves\...`.

This matters because manual save repair can appear to fail when in reality only one of the mirrored save locations was edited.

## Fast Iteration Loop

Use this loop:

1. edit managed code
2. rebuild the `.dll`
3. rebuild or refresh the `.pck` if assets or localization changed
4. copy the final payload into `mods\`
5. launch the game
6. inspect logs immediately if the mod does not appear or does not behave as expected

Do not skip `.pck` refresh when the change touched assets, localization, or Godot import outputs.

The point of this loop is to keep code and content changes synchronized. STS2 mods commonly split behavior between a managed assembly and a packed Godot payload. If one of those layers is refreshed and the other is stale, the resulting symptom can be misleading.

## Separate Code And Content Pipelines

Treat these as different failure domains:

- `.dll` pipeline: compile, references, initializer, Harmony patches
- `.pck` pipeline: manifest, assets, localization, import artifacts

Code can compile while strings fail to load. Assets can exist while initialization never runs.

A useful debugging habit is to ask which pipeline owns the current failure before trying to fix the failure itself.

## Logging Strategy

Add explicit log points around:

- mod initialization
- patch registration
- content registration
- localization loading
- the specific gameplay event you are intercepting

This is not redundant with game logging. The game can tell you that a payload loaded, but only the mod can tell you which branch of its own logic executed, which registration completed, or whether a key patch site was reached.

## High-Signal Failure Patterns

### Mod does not appear at all

Usually one of:

- wrong install path
- missing or malformed `<ModId>.json`
- missing or malformed payload manifest
- bad `pck_name`
- broken initializer target
- version mismatch

This is the classic "nothing happened" category, and it should usually be debugged from the outside inward: installation shape, metadata, payload contract, initializer, then version assumptions.

### Strings missing even though the mod loaded

Usually one of:

- stale `.pck`
- wrong internal localization path
- wrong language code
- wrong table or key names

This is the classic "the code loaded but the content layer did not" category.

### Card logic fires but the card lifecycle breaks

Usually one of:

- patching too early in the play flow
- wrapping the wrong `OnPlay(...)` task chain

Move post-resolution behavior to a safer hook such as `AfterCardPlayedLate` unless you truly need to own the card's internal play logic.

This is a good reminder that a gameplay effect and a gameplay lifecycle are not the same thing.

### `Sequence contains no matching element` from `RelicModel.get_Pool()`

Usually means the relic is being rendered in inspect or hover UI without a valid pool association. Fix the pool path before chasing unrelated UI code.

This is one of the highest-value error signatures because it collapses a large search space into a specific content contract.

## Version Mismatch Is Normal

Suspect a version mismatch when:

- a previously valid signature no longer resolves
- a hook changes shape
- asset or localization conventions shift after a game update

STS2 is still in Early Access. Revalidation is part of normal maintenance.
