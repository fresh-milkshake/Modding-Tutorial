# Slay the Spire 2 Modding Handbook

This handbook describes the current local modding contract of Slay the Spire 2 and then applies it to one coherent example containing a card, potion, relic, map event, and custom overlay. The game is the primary source: the installed assembly defines behavior, while the main PCK defines resource paths and localization conventions.

## Source Hierarchy

1. `release_info.json`, `sts2.runtimeconfig.json`, and the installed assemblies
2. decompiled implementation in `sts2.dll`
3. resources and localization in `SlayTheSpire2.pck`
4. runtime logs and a clean installation test
5. observed release packages and community documentation

This order matters during Early Access. A working pattern from an older release is evidence of what once worked, not proof of the current contract.

## Current Baseline

| Item | Verified value |
| --- | --- |
| STS2 version | `v0.103.3` |
| Commit | `460a0ece` |
| Release metadata date | `2026-05-29T13:36:05-07:00` |
| Target framework | `net9.0` |
| Bundled .NET runtime | `9.0.7` |
| Main PCK format | Godot `4.5.1` |

The current loader recursively scans `<STS2>/mods` for JSON manifests. Each manifest owns the directory that contains it and declares whether sibling `<id>.dll` and `<id>.pck` files must be loaded. The recommended installation is therefore:

```text
mods/
  MyMod/
    MyMod.json
    MyMod.dll
    MyMod.pck
```

The PCK is optional for code-only mods. There is no second embedded manifest contract in `v0.103.3`.

## Foundation

1. [01. Environment Setup](01-environment-setup.md)
2. [02. Game Layout And Loader Surface](02-vanilla-sts2-layout.md)
3. [03. Project Skeleton, Build, And Manifest](03-mod-loading-and-manifests.md)
4. [04. Reverse-Engineering The Game](04-reverse-engineering-the-game.md)
5. [05. Behavior Mods, Harmony, And Hooks](05-entrypoints-harmony-and-hooks.md)
6. [06. Content Infrastructure, Assets, And Localization](06-assets-pck-and-localization.md)
7. [07. Native UI And Runtime Inspection](07-native-ui-and-runtime-inspection.md)
8. [08. Debugging And Fast Iteration](08-debugging-and-fast-iteration.md)
9. [09. Packaging, Installing, And Verifying](09-packaging-installing-and-verifying.md)

## Applied Content Tutorial

The following chapters build a small fictional content set called `FieldNotes`. The name is only a stable namespace for examples; each chapter can be used independently.

10. [10. Custom Cards](10-custom-cards.md)
11. [11. Custom Potions](11-custom-potions.md)
12. [12. Custom Relics](12-custom-relics.md)
13. [13. Custom Act Events](13-custom-act-events.md)
14. [14. Custom GUI](14-custom-gui.md)

The sequence deliberately repeats the same lifecycle:

1. define a concrete `AbstractModel` subtype
2. let `ModelDb` discover the type from the loaded assembly
3. place it in the game system that can select it
4. supply localization and resources under the paths the model resolves
5. test gameplay, inspect, hover, unlock, and save/load behavior separately

## Reference

- [Appendix A: API Inventory](appendix-api-inventory.md)
- [Appendix B: Release Package Patterns](appendix-example-mods.md)
- [References](references.md)

The tutorial assumes basic familiarity with C#, async methods, Harmony attributes, and the Godot scene tree. It focuses on STS2-specific contracts rather than general language instruction.
