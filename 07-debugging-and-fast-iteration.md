# 07. Debugging And Fast Iteration

## Start With The Vanilla Signals

Before adding custom helpers, use the signals the game already gives you:

- `sts2_stdout.log`
- `sts2_stderr.log`
- the presence or absence of your mod under the mod loader UI
- behavior changes in-game

Useful log facts already visible in this install:

- engine version line
- release version line
- locale loading lines
- save path information

If your mod does not initialize, the logs are usually the first place to check.

For current local debugging, also check:

- `%APPDATA%\SlayTheSpire2\logs\godot.log`
- `%APPDATA%\SlayTheSpire2\logs\godot<timestamp>.log`

Those Godot-side logs can contain managed exception traces that never show up in `sts2_stdout.log`.

## What To Verify First When A Mod Fails To Load

1. does the mod directory contain the expected `.dll` and `.pck`?
2. is the manifest present in the expected place?
3. does `pck_name` match the mod payload naming?
4. is your `[ModInitializer]` attribute on a real type and pointing at a real method?
5. are you building against the same STS2 version you are running?

## Fast Iteration Loop

A practical STS2 iteration loop looks like this:

1. edit managed code
2. rebuild the `.dll`
3. rebuild or update the `.pck` if assets/localization changed
4. copy payload into the local `mods/` target
5. launch the game
6. inspect logs immediately if the mod does not appear or does not behave as expected

Do not skip step 3. A lot of "my code works but strings/assets do not" failures are just stale `.pck` payloads.

## Separate Code Iteration From Content Iteration

Treat these as two independent pipelines:

- `.dll` pipeline: C# compile, reference resolution, Harmony patch validity, initializer validity
- `.pck` pipeline: manifest, assets, localization, Godot import/export state

This split helps because many STS2 failures only affect one half:

- code can compile while strings still fail to load
- assets can be present while initialization never runs

## Debugging Managed Code

For code-level debugging, your options are usually:

- heavy decompilation and log-driven debugging
- debugger attachment through a custom startup/debug setup
- local helper tooling

The correct amount of sophistication depends on the mod.

For small content mods, logs plus targeted Harmony patches are often enough.

For UI-heavy or systems-heavy mods, you usually want a stronger iteration setup.

## Repo-Local Helper Tooling

This install contains local helper tooling that is useful, but not vanilla:

- `tools/bin/godotpcktool.exe`
- `tools/ilspy/...`
- `tools/sts2-debug-hook/...`
- `tools/sts2-loose-mod/...`

Treat these as **optional accelerators** for this repo, not as prerequisites for STS2 modding in general.

### `sts2-debug-hook`

The local helper README describes it as:

- injecting a startup hook without modifying the game exe or main pack
- patching early Godot/C# UI bootstrap points with Harmony
- adding an always-on-top debug panel

This is useful for deep local investigation, but it is not part of the base game and should be documented as a custom workflow.

### `sts2-loose-mod`

The local helper sample demonstrates:

- compiling a minimal mod
- producing a `.dll`
- creating a `.pck`
- writing a manifest

That makes it a useful local reference project, but again, it is repo-local tooling rather than STS2 canonical documentation.

## Logging Strategy For Your Own Mod

Even if you use a debugger, add explicit log points around:

- mod initialization
- successful patch registration
- content registration
- localization table loading
- key game events you are intercepting

Without that, many STS2 failures collapse into "the mod didn't seem to do anything".

## High-Value Debug Questions

When something breaks, narrow it fast:

- did the mod load at all?
- did the initializer run?
- did Harmony patch registration succeed?
- did content registration happen?
- did the game find the assets?
- did the game find the strings?

These are different failures. Do not debug them as one giant problem.

## High-Signal Error Signatures

Some failures are distinctive enough that you should jump straight to the likely cause:

### `Sequence contains no matching element` from `RelicModel.get_Pool()`

Typical implication:

- a custom relic is being rendered in hover or inspect UI
- `DynamicDescription` went through `EnergyIconHelper`
- the relic is not present in any `RelicPool`

Likely fix:

- assign or patch a meaningful pool for the relic, even if it is start-only and should not appear in normal rewards

### Strings missing even though the mod loaded

Typical implication:

- the `.dll` initialized correctly
- the `.pck` exists
- the localization JSON is packed under the wrong internal path

Likely fix:

- inspect the pack and confirm the file lives at `res://<pck_name>/localization/<lang>/<file>`

### Relic icon did not visually update after changing the PNG

Typical implication:

- the source PNG changed
- the `.ctex` or related import artifacts in the pack are stale

Likely fix:

- refresh Godot imports and repack the current `.ctex`, `.md5`, and `.png.import` artifacts together with the source image

## When To Suspect A Version Mismatch

Suspect version mismatch when:

- your mod compiled yesterday but fails after a game patch
- a previously valid method signature no longer resolves
- a hook you depended on changed shape
- assets or localization conventions appear to shift between builds

Because STS2 is still in Early Access, this is not an edge case. It is normal maintenance.
