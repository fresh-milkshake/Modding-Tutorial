# UndoAndRedo

Archive: `UndoAndRedo-16-1-0-4-1773092418.zip`

## Extracted Tree

```text
UndoAndRedo/
  UndoAndRedo.dll
  UndoAndRedo.pck
```

## Layout Classification

- archive shape: root payload
- install implication: place these files inside a mod folder under `mods\`

## Embedded Manifest

```json
{
  "pck_name": "UndoAndRedo",
  "name": "UndoAndRedo",
  "author": "JiesiLuo",
  "description": "Press Left/Right Arrow to undo/redo combat actions. Inspired by filippobaroni's Undo the Spire mod for STS1.",
  "version": "1.0.0"
}
```

## PCK Notes

- pack version: `2`
- Godot version: `4.5.1`
- embedded `mod_manifest.json`
- no loose manifest
- extremely small `.pck`, effectively manifest-only

## What This Example Teaches

- a mod can keep almost all of its logic in the `.dll` and use the `.pck` as a very thin metadata container
- root-payload release zips remain common even when the mod itself is otherwise cleanly packaged
