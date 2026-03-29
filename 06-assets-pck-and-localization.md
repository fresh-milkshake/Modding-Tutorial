# 06. Content Mods, Assets, PCK, And Localization

## Start With Model Registration

When you are adding new cards, relics, powers, or other discoverable content, inspect the model-registration path before writing invasive patches.

Useful entry points:

- `ModHelper.AddModelToPool`
- related pool classes
- model database lookup paths

If registration already has a supported path, prefer that over patching random generation code.

This is one of the main dividing lines between a maintainable content mod and a brittle one. A content mod that enters through the same conceptual doors as the base game tends to stay legible even as internal implementation details change. A content mod that begins by forcing itself into downstream generation code often works only as long as every hidden assumption remains true.

## Content-Mod Checklist

Before you ship a new model, verify:

1. it is discoverable from the right pool or lookup path
2. its localization keys match the table the game actually reads
3. its icon names and paths match the files you packed
4. unlock-state exposure and catalog visibility are handled
5. inspect and hover UI can resolve any metadata they need

These checks belong together because STS2 content is not introduced by a single event. A model may need to exist in gameplay systems, in generation or pool systems, in unlock-state systems, and in UI rendering systems. A mod can therefore be "partly correct" in several different ways: registered but not visible, visible but not inspectable, inspectable but missing strings, or fully localized but still missing a valid pool path.

## Custom Relic Contracts Worth Verifying

For relics in the current build:

- `Title`, `Description`, and `Flavor` default to keys derived from `Id.Entry`
- `IconBaseName` defaults to `Id.Entry.ToLowerInvariant()`
- `BigIconPath` points at `res://images/relics/<IconBaseName>.png`
- `DynamicDescription` can depend on `RelicModel.Pool`

Practical consequence:

- a relic can render a small icon but still break large-art or inspect flows if `IconBaseName` and packed files do not match
- a start-only or special-case relic can still need a meaningful pool association for hover and inspect code paths

This is a good example of how STS2 content contracts extend beyond a single class override. A relic model is not merely a gameplay object. It is also a localization anchor, an icon naming anchor, and in some UI paths a pool-aware object. If one of those assumptions is left unresolved, the failure often surfaces far away from the original code change.

## Unlock Visibility And Catalog Visibility

Do not assume one integration point covers both:

- making content exist in the game
- making content visible in unlock state or compendium-style UI

In practice, content mods may need separate handling for:

- pool membership
- unlock-state getters
- catalog-entry visibility

The practical lesson is that content visibility is distributed. A model can be perfectly legitimate in gameplay and still remain absent from a compendium, hidden behind unlock logic, or filtered by a visibility getter. The safest workflow is to ask separately: how does the game know this model exists, and under what conditions does the game choose to show it?

## What The Vanilla Pack Confirms

`SlayTheSpire2.pck` confirms:

- vanilla localization lives under `localization/<lang>/...`
- relic art uses Godot-imported resources such as `images/relics/*.png.import`
- vanilla string tables include files such as `cards.json`, `characters.json`, `relics.json`, and `static_hover_tips.json`

Vanilla language codes include `eng`, `rus`, `zhs`, and others documented in `localization/README.md`.

This matters because content mods inherit both behavior and vocabulary from the shipped game. The pack does not merely provide examples; it provides the naming system the game already uses. Matching that naming system usually lowers the amount of custom translation code a mod must own.

## Modded Localization Path

For the current build, `GetModdedLocTables(language, file)` looks for:

```text
res://<pck_name>/localization/<language>/<file>
```

Safe rule:

- keep vanilla table names such as `relics.json`
- pack them under `res://<pck_name>/localization/<lang>/...`
- do not assume a flat `res://localization/...` layout will be merged for mods just because vanilla uses it

This is one of the easiest places for a mod to look almost correct while still failing at runtime. A localization JSON file can exist, contain valid keys, and even resemble the vanilla folder layout, but if it is packed under the wrong internal path the loader will never merge it. For that reason, internal pack paths are more important than external filenames.

## What Can Live In A Mod `.pck`

A mod `.pck` can contain:

- `mod_manifest.json`
- localization JSON
- icon assets
- Godot import metadata
- `project.binary`
- other Godot-side resources

It is a general Godot payload container, not just an asset folder.

That is why two working STS2 mods can have radically different `.pck` sizes while both remaining valid. Some payloads need little more than a manifest. Others need art, localization, imported textures, and additional Godot metadata. The file extension alone does not tell you how substantial the content layer really is.

## PCK Tooling

To inspect or build packs, you need a Godot-compatible PCK tool.

Repo-local helpers such as `godotpcktool.exe` are useful, but they are optional. The key rule is not the tool choice; it is verifying the internal pack paths after packing.

A successful pack operation only proves that a file was produced. It does not prove that the game will interpret the resulting internal layout the way you intended.

## Imported Texture Artifacts Matter

For image changes, the source `.png` is not the whole story. Stale Godot import outputs can leave old visuals in place.

When you update art, verify the current pack includes the refreshed import artifacts together with the source image.

In practical terms, visual bugs that look like cache issues often are cache issues, but they are Godot import-cache issues expressed through a packed payload.
