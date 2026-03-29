# BetterSpire2

Archive: `BetterSpire2-2-1-5-1773086303.zip`

## Extracted Tree

```text
BetterSpire2/
  BetterSpire2.dll
  BetterSpire2.pck
```

## Layout Classification

- archive shape: root payload
- current local install implication for `v0.99.1`: normalize this archive into root-level installed files under `mods\`, not just a folder payload

## Embedded Manifest

```json
{
  "pck_name": "BetterSpire2",
  "name": "Better Spire 2",
  "author": "JDR",
  "description": "A minimal QOL mod pack. Features: Multi-hit damage totals per enemy, total incoming damage above player, hold R to restart run, skip splash screen. Press F1 for settings.",
  "version": "1.0.0"
}
```

## PCK Notes

- pack version: `2`
- Godot version: `4.3.0`
- embedded `mod_manifest.json`
- includes icon assets and `project.binary`
- no loose manifest next to the payload

## What This Example Teaches

- STS2 can encounter real-world mods packaged from an older Godot minor line
- simple code-focused mods may use a very light `.pck` containing little more than manifest and project/icon metadata
- root-level archive payloads are common, but on the current local build you should normalize them into root-level installed files under `mods\`
