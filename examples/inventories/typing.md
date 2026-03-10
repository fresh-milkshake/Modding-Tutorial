# typing

Archive: `typing.zip-9-0-0-3-1773063286.zip`

## Extracted Tree

```text
typing/
  typing.dll
  typing.pck
```

## Layout Classification

- archive shape: root payload
- install implication: place these files inside a mod folder under `mods\`

## Embedded Manifest

```json
{
  "pck_name": "typing",
  "name": "sts2_typing",
  "author": "shiroi",
  "description": "support",
  "version": "0.0.1"
}
```

The extracted manifest text includes inline comments in the original file, which suggests the author exported a developer-oriented manifest rather than a polished release manifest.

## PCK Notes

- pack version: `3`
- Godot version: `4.5.1`
- embedded `mod_manifest.json`
- includes `Scripts/Entry.cs`
- includes icon assets and `project.binary`

## What This Example Teaches

- manifests inside a `.pck` may contain rough author-side metadata and still exist in a working release
- small code mods may ship minimal Godot-side structure
- not every public STS2 mod archive is polished; your own installer instructions should be stricter than what random examples do
