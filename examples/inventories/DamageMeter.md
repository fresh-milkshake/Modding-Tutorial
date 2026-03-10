# DamageMeter

Archive: `DamageMeter-12-0-1-2-1773055212.zip`

## Extracted Tree

```text
DamageMeter/
  CHANGELOG.txt
  README.txt
  mods/
    DamageMeter/
      DamageMeter.dll
      DamageMeter.pck
      mod_manifest.json
```

## Layout Classification

- archive shape: installer-friendly `mods/<Name>/...`
- install implication: copy the included `mods\DamageMeter` folder into the game root

## Loose Manifest

```json
{
  "pck_name": "DamageMeter",
  "name": "Damage Meter",
  "author": "",
  "description": "显示输出、真实伤害、能量与复盘图表 / Show damage, true damage, energy, and review charts.",
  "version": "0.1.2"
}
```

## PCK Notes

- pack version: `3`
- Godot version: `4.5.1`
- loose and embedded `mod_manifest.json`
- contains `DamageMeter.localization.en.json`
- contains `DamageMeter.localization.zhs.json`
- contains many source `.cs` files
- contains `.godot` metadata, imported icon data, and `project.binary`

## README Notes

The bundled README explicitly documents:

- local installation by copying `mods\DamageMeter`
- HUD controls
- settings persistence outside the mod folder
- a separate settings JSON under `%APPDATA%\Godot\app_userdata\DamageMeter\`

## What This Example Teaches

- this is the best real-world packaging example in the set
- loose manifest plus embedded manifest is a valid release strategy
- mods can ship their own localization files without mirroring the exact vanilla pack tree
- STS2 mods can persist settings outside the game root using Godot's user data conventions
