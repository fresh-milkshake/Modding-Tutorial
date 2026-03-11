# 06. Assets, PCK, And Localization

## What The Vanilla Pack Confirms

`SlayTheSpire2.pck` directly confirms these vanilla content conventions:

- localization lives under `localization/<lang>/...`
- the pack includes `localization/README.md`
- English source tables include files such as:
  - `localization/eng/cards.json`
  - `localization/eng/characters.json`
  - `localization/eng/relics.json`
  - `localization/eng/static_hover_tips.json`
- relic assets use `images/relics/*.png.import`

Example:

- `images/relics/akabeko.png.import`

The imported file metadata confirms these are ordinary Godot-imported resources.

## Language Codes

The vanilla `localization/README.md` explicitly describes the language folders using short codes such as:

- `eng`
- `deu`
- `esp`
- `ita`
- `jpn`
- `kor`
- `pol`
- `ptb`
- `rus`
- `tha`
- `tur`
- `zhs`

That README also confirms the base game expects JSON files with string keys and string values.

## Asset Naming

Vanilla relic asset names are snake_case:

- `akabeko`
- `art_of_war`
- `bag_of_marbles`
- `burning_blood`

That aligns with the lamali guide's recommendation to use snake_case filenames for relic images.

Safe rule:

- if your content maps to a model name, assume the asset filename should be snake_case unless your verified implementation says otherwise

## What Goes Inside A Mod `.pck`

Real example packs show that mod `.pck` files may contain:

- `mod_manifest.json`
- `project.binary`
- `.godot/` caches
- imported textures
- icon files
- source `.cs` files
- custom localization JSON

Examples:

- `UpgradeAllCards.pck` only contains `mod_manifest.json`
- `BetterSpire2.pck` contains `mod_manifest.json`, icon assets, and `project.binary`
- `DamageMeter.pck` contains source files, localization files, icon assets, `.godot` metadata, and `project.binary`

This means `.pck` is not "assets only" in practice. It is a general Godot-side payload container.

## Localization: One Important Caveat

The lamali guide presents a Godot-side layout that mirrors vanilla localization folders. That is a sensible and well-explained convention.

However, the real example mods show more than one working pattern.

Observed in `DamageMeter.pck`:

- `DamageMeter.localization.en.json`
- `DamageMeter.localization.zhs.json`

Observed in vanilla STS2:

- `localization/eng/relics.json`
- many other `localization/<lang>/*.json` files

However, the current loader implementation is stricter than that high-level observation might suggest.

Verified in the current build:

- `ModManager.GetModdedLocTables(language, file)` looks for `res://<pck_name>/localization/<language>/<file>`
- that means a modded `relics.json` intended to merge into the base `relics` table should be packed under a folder named after the manifest `pck_name`
- example: if `pck_name` is `VenomousDeity`, the working path is `res://VenomousDeity/localization/eng/relics.json`

Therefore, do **not** document a single universal mod localization layout unless you have verified it against the current loader implementation, and do not assume that a flat `res://localization/...` mod path will be merged just because vanilla uses it.

Safe statement:

- STS2 can gather modded localization tables
- the vanilla game uses `localization/<lang>/...`
- the current modded merge path is `res://<pck_name>/localization/<lang>/<file>`
- community and real-world mods may still include other localization JSON layouts for their own internal systems or UI

## Recommended Practical Localization Strategy

For a new mod, the safest documented path is:

1. keep your base table names aligned with vanilla, for example `relics.json`
2. pack them under `res://<pck_name>/localization/<lang>/...`
3. validate your exact layout against a current game build before you scale it up
4. compare against a known working mod if your strings do not load

If you are debugging localization problems, compare all three sources:

- your mod pack contents
- `GetModdedLocTables`-related game behavior
- a working mod such as `DamageMeter`

## BBCode And Rich Text

The vanilla localization README documents BBCode-style formatting such as:

- `[b]...[/b]`
- `[i]...[/i]`
- `[color=X]...[/color]`
- `[gold]...[/gold]`
- `[purple]...[/purple]`
- `[jitter]...[/jitter]`
- `[sine]...[/sine]`

That matters because STS2 text is not plain string substitution. If you add custom strings, preserve the game's formatting conventions.

## `.pck` Tooling

To inspect or build packs you need a Godot-compatible PCK tool.

In this repo, the local helper tool is:

```text
tools/bin/godotpcktool.exe
```

It supports:

- `list`
- `extract`
- `add`
- `repack`

Important: this tool is convenient in this repo, but it is not part of vanilla STS2 by default.

## Packing Example With The Repo-Local Tool

If you use the repo-local helper packer, the command shape is:

```powershell
godotpcktool.exe `
  --pack MyMod.pck `
  --action add `
  --set-godot-version 4.5.1 `
  --remove-prefix "D:\path\to\godot-export-root\" `
  --file "D:\path\to\godot-export-root\mod_manifest.json" `
  --file "D:\path\to\godot-export-root\project.binary" `
  --file "D:\path\to\godot-export-root\images\..." `
  --file "D:\path\to\godot-export-root\MyMod\localization\eng\relics.json"
```

Interpretation:

- `--pack` chooses the target `.pck`
- `--action add` creates or appends content
- `--set-godot-version 4.5.1` matches the current game baseline
- `--remove-prefix` strips your local export root so the paths inside the pack are relative

Always inspect the resulting pack after building it. A `.pck` that exists is not automatically a `.pck` with the right internal paths.

For localization specifically, inspect the internal pack path, not just the filename on disk. A modded `relics.json` in the wrong folder can exist in the `.pck` and still never be merged by the game.

## Imported Texture Artifacts Matter

For relic art, the `.png` file is not the whole story.

In real debugging, stale Godot import outputs such as:

- `.godot/imported/<name>.ctex`
- `.godot/imported/<name>.md5`
- `images/relics/<name>.png.import`

can cause a relic icon to stay visually old even when the source PNG changed.

Safe rule:

- if you changed a relic image, rebuild or refresh the imported Godot artifacts before repacking the `.pck`
- when verifying the pack, confirm the current `.ctex` and related import metadata are included alongside the source `.png`

## Packaging Guidance

Keep your mod `.pck` focused on Godot-side content:

- manifest
- assets
- localization
- optional Godot project metadata

Keep your gameplay logic in the compiled `.dll`.

Some real mods also ship source `.cs` files inside the `.pck`. STS2 can load such mods, but that is not required and is usually not necessary for release.
