# 08. Debugging And Fast Iteration

## Start With Native Signals

Before adding custom tooling, check:

- `sts2_stdout.log`
- `sts2_stderr.log`
- the mod loader UI
- in-game behavior

Also check Godot-side logs under `%APPDATA%\SlayTheSpire2\logs\...`. They can contain managed traces that never appear in `sts2_stdout.log`.

## Save Paths Worth Knowing

For run-state debugging on this machine, active saves can exist in both:

- `%APPDATA%\SlayTheSpire2\steam\<SteamId>\...`
- `C:\Program Files (x86)\Steam\userdata\<SteamAccountId>\2868840\remote\...`

That includes modded runs under `modded\profile1\saves\...`.

## Fast Iteration Loop

Use this loop:

1. edit managed code
2. rebuild the `.dll`
3. rebuild or refresh the `.pck` if assets or localization changed
4. copy the final payload into `mods\`
5. launch the game
6. inspect logs immediately if the mod does not appear or does not behave as expected

Do not skip `.pck` refresh when the change touched assets, localization, or Godot import outputs.

## Separate Code And Content Pipelines

Treat these as different failure domains:

- `.dll` pipeline: compile, references, initializer, Harmony patches
- `.pck` pipeline: manifest, assets, localization, import artifacts

Code can compile while strings fail to load. Assets can exist while initialization never runs.

## Logging Strategy

Add explicit log points around:

- mod initialization
- patch registration
- content registration
- localization loading
- the specific gameplay event you are intercepting

## High-Signal Failure Patterns

### Mod does not appear at all

Usually one of:

- wrong install path
- missing or malformed `<ModId>.json`
- missing or malformed payload manifest
- bad `pck_name`
- broken initializer target
- version mismatch

### Strings missing even though the mod loaded

Usually one of:

- stale `.pck`
- wrong internal localization path
- wrong language code
- wrong table or key names

### Card logic fires but the card lifecycle breaks

Usually one of:

- patching too early in the play flow
- wrapping the wrong `OnPlay(...)` task chain

Move post-resolution behavior to a safer hook such as `AfterCardPlayedLate` unless you truly need to own the card's internal play logic.

### `Sequence contains no matching element` from `RelicModel.get_Pool()`

Usually means the relic is being rendered in inspect or hover UI without a valid pool association. Fix the pool path before chasing unrelated UI code.

## Version Mismatch Is Normal

Suspect a version mismatch when:

- a previously valid signature no longer resolves
- a hook changes shape
- asset or localization conventions shift after a game update

STS2 is still in Early Access. Revalidation is part of normal maintenance.
