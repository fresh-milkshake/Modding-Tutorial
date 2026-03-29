# 06. Content Mods, Assets, PCK, And Localization

## Start With Model Registration

When you are adding new cards, relics, powers, or other discoverable content, inspect the model-registration path before writing invasive patches.

Useful entry points:

- `ModHelper.AddModelToPool`
- related pool classes
- model database lookup paths

If registration already has a supported path, prefer that over patching random generation code.

## Content-Mod Checklist

Before you ship a new model, verify:

1. it is discoverable from the right pool or lookup path
2. its localization keys match the table the game actually reads
3. its icon names and paths match the files you packed
4. unlock-state exposure and catalog visibility are handled
5. inspect and hover UI can resolve any metadata they need

## Custom Relic Contracts Worth Verifying

For relics in the current build:

- `Title`, `Description`, and `Flavor` default to keys derived from `Id.Entry`
- `IconBaseName` defaults to `Id.Entry.ToLowerInvariant()`
- `BigIconPath` points at `res://images/relics/<IconBaseName>.png`
- `DynamicDescription` can depend on `RelicModel.Pool`

Practical consequence:

- a relic can render a small icon but still break large-art or inspect flows if `IconBaseName` and packed files do not match
- a start-only or special-case relic can still need a meaningful pool association for hover and inspect code paths

## Unlock Visibility And Catalog Visibility

Do not assume one integration point covers both:

- making content exist in the game
- making content visible in unlock state or compendium-style UI

In practice, content mods may need separate handling for:

- pool membership
- unlock-state getters
- catalog-entry visibility

## What The Vanilla Pack Confirms

`SlayTheSpire2.pck` confirms:

- vanilla localization lives under `localization/<lang>/...`
- relic art uses Godot-imported resources such as `images/relics/*.png.import`
- vanilla string tables include files such as `cards.json`, `characters.json`, `relics.json`, and `static_hover_tips.json`

Vanilla language codes include `eng`, `rus`, `zhs`, and others documented in `localization/README.md`.

## Modded Localization Path

For the current build, `GetModdedLocTables(language, file)` looks for:

```text
res://<pck_name>/localization/<language>/<file>
```

Safe rule:

- keep vanilla table names such as `relics.json`
- pack them under `res://<pck_name>/localization/<lang>/...`
- do not assume a flat `res://localization/...` layout will be merged for mods just because vanilla uses it

## What Can Live In A Mod `.pck`

A mod `.pck` can contain:

- `mod_manifest.json`
- localization JSON
- icon assets
- Godot import metadata
- `project.binary`
- other Godot-side resources

It is a general Godot payload container, not just an asset folder.

## PCK Tooling

To inspect or build packs, you need a Godot-compatible PCK tool.

Repo-local helpers such as `godotpcktool.exe` are useful, but they are optional. The key rule is not the tool choice; it is verifying the internal pack paths after packing.

## Imported Texture Artifacts Matter

For image changes, the source `.png` is not the whole story. Stale Godot import outputs can leave old visuals in place.

When you update art, verify the current pack includes the refreshed import artifacts together with the source image.
