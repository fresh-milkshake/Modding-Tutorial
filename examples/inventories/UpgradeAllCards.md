# UpgradeAllCards

Archive: `UpgradeAllCards-5-1-0-0-1772940988.zip`

## Extracted Tree

```text
UpgradeAllCards/
  UpgradeAllCards.dll
  UpgradeAllCards.pck
```

## Layout Classification

- archive shape: root payload
- current local install implication for `v0.99.1`: normalize this archive into root-level installed files under `mods\`, for example `UpgradeAllCards.json`, `UpgradeAllCards.dll`, and `UpgradeAllCards.pck`

## Embedded Manifest

```json
{
  "pck_name": "UpgradeAllCards",
  "name": "Upgrade All Cards",
  "author": "JiesiLuo",
  "description": "Automatically upgrades all starting cards and any new cards added to your deck during a run.",
  "version": "1.0.0"
}
```

## PCK Notes

- pack version: `2`
- Godot version: `4.5.1`
- embedded `mod_manifest.json`
- no loose manifest
- no visible asset payload beyond the embedded manifest

## Verified Initializer Use

The compiled assembly contains:

- `UpgradeAllCards.UpgradeAllCardsMod => ModInitializer("Initialize")`

That makes this mod one of the cleanest real examples of the type-level initializer pattern used by STS2.

## What This Example Teaches

- this is the smallest clean starter example in the set
- a functioning STS2 mod can have a near-empty `.pck` when it has no meaningful Godot-side assets
- the simplest real mod release can still teach you the correct manifest and initializer pattern
