# 08. Debugging And Fast Iteration

Debug STS2 mods from the loader inward. The first failed layer usually invalidates evidence from every later layer.

## Log Sources

Check both game-root and Godot user logs:

- `<STS2>/sts2_stdout.log`
- `<STS2>/sts2_stderr.log`
- `%APPDATA%/SlayTheSpire2/logs/...`
- the in-game mod loader status and error display

The root logs can be stale if the game has not recently written them. Compare timestamps before treating their version header as current.

## Iteration Loop

1. close the game when replacing loaded DLLs or PCKs
2. build the DLL against the current installed assemblies
3. rebuild the PCK only when resources changed, but never reuse a stale pack accidentally
4. recreate the staged mod directory from known outputs
5. copy the whole directory to `<STS2>/mods/<ModId>/`
6. launch and inspect loader messages before testing gameplay
7. test the smallest path that exercises the change

Staging should be reproducible. Incrementally copying individual files leaves obsolete resources and manifests behind.

## Loader Failure Signatures

### Manifest not detected

Check:

- the file has a `.json` extension below `<STS2>/mods`
- the JSON is valid
- `id` is present
- player consent and disabled-mod state permit loading

### DLL or PCK reported missing

The loader resolves payloads beside the manifest using the exact `id`:

```text
<manifest directory>/<id>.dll
<manifest directory>/<id>.pck
```

Check `has_dll`, `has_pck`, case, and basenames.

### Dependency failure

Check that every dependency string equals another manifest's `id`. Display names and assembly names are not dependency identifiers. Inspect logs for missing or circular chains.

### Initializer failure

Check:

- the attributed type is present in the loaded DLL
- the named method exists and is static
- registration does not enumerate a pool before all additions are complete
- the exception is not wrapped by reflection in `TargetInvocationException`

If no initializer exists, remember that the loader automatically calls Harmony `PatchAll`; an unexpected patch can therefore originate from an assembly that appears to have no entry point.

## Content Failure Signatures

### `Sequence contains no matching element` from `PotionModel.Pool` or `RelicModel.Pool`

The model exists but belongs to no current pool. Register it with `ModHelper.AddModelToPool` before initialization freezes the pool.

### Model can be created but never appears naturally

Discovery and selection are separate. Verify pool membership for cards, potions, and relics. For events, verify the target act or shared event list is patched.

### Missing localization

Check the PCK for:

```text
<id>/localization/<language>/<table>.json
```

Then check the model slug, table name, flat JSON key, and language code. The manifest ID controls the prefix.

### Missing image or invalid cast to `Texture2D`

Check whether the model expects a PNG or a `.tres` texture resource. Potion art uses a fixed atlas-resource path; placing only a namespaced PNG does not satisfy that lookup.

### Card effect runs but card flow breaks

The effect is attached too early or owns the wrong async chain. Prefer the model's `OnPlay` for a custom card and `AfterCardPlayedLate` for external post-resolution behavior.

## Save-State Troubleshooting

Active saves may exist under both:

- `%APPDATA%/SlayTheSpire2/steam/<SteamId>/...`
- `<Steam>/userdata/<AccountId>/2868840/remote/...`

Modded runs use modded profile save paths. Before editing or deleting a save, close the game and preserve both copies. Cloud synchronization can restore the copy that was not changed.

Custom models serialize by model ID through the game's save types. Renaming a model class changes its generated entry ID unless compatibility code maps the old ID. Treat class names as persistent data once a release can appear in a save.

## Instrumentation

Log high-value boundaries:

- initializer entered and completed
- each pool registration
- each Harmony patch group
- event list augmentation
- GUI controller attachment
- model effect entry with owner and state identifiers

Avoid per-frame logs. They hide the first meaningful error and make UI debugging harder.

## Update Regression Matrix

After a game update, test:

1. loader detection
2. new run creation
3. every custom content type through natural generation
4. hover, inspect, and library views
5. save and reload with custom content owned
6. event entry and completion
7. GUI open, close, focus, and tooltip ordering

This matrix detects semantic drift that a successful build cannot prove absent.
