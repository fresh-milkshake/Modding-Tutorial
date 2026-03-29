# BetterSovereignBlade

Archive: `BetterSovereignBlade-17-1-0-0-1773072075.zip`

## Extracted Tree

```text
BetterSovereignBlade/
  BetterSovereignBlade/
    BetterSovereignBlade.dll
    BetterSovereignBlade.pck
```

## Layout Classification

- archive shape: subfolder payload
- archive install implication: extract the included `BetterSovereignBlade` folder
- current local install implication for `v0.99.1`: normalize the final installed payload into root-level files under `mods\` if the folder-only shape is not recognized

## Embedded Manifest

```json
{
  "pck_name": "BetterSovereignBlade",
  "name": "BetterSovereignBlade",
  "author": "Emo Used HM01",
  "description": "Reduces Sovereign Blade's base energy cost from 2 to 1",
  "version": "v0.0"
}
```

## PCK Notes

- pack version: `3`
- Godot version: `4.5.1`
- embedded `mod_manifest.json`
- includes icon assets and a `mod_image.png.import`
- includes `project.binary`
- includes near-empty `.cs` files inside the pack

## What This Example Teaches

- a mod can be packaged as a folder-first release instead of a root-level archive
- a manifest inside the `.pck` is enough for a working release shape
- even a very small gameplay tweak may still ship a Godot-side icon/image payload
