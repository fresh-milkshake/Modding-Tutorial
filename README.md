# Slay the Spire 2 Modding Tutorial

This repository is a version-aware tutorial for building <u>local-only</u> C# mods for **Slay the Spire 2 Early Access**. It is written for readers who already know the basics of C#, .NET, and Godot, and want a compact explanation of how those tools meet the concrete loading, packaging, and runtime contracts used by the live game.

## Covers

- setup and verified version baseline
- game layout and built-in mod loader surface
- project skeleton, `.csproj`, manifests, and staging layout
- reverse engineering with `sts2.dll`, `SlayTheSpire2.pck`, and logs
- `[ModInitializer]`, Harmony, hooks, and safer patch selection
- content registration, pools, localization, icons, and unlock visibility
- native UI reuse with submenu and overlay screens
- debugging, packaging, installation, and verification

The tutorial is intentionally structured around the actual lifecycle of a mod. It begins with the local game baseline, moves through loader and payload structure, then splits into behavior mods, content mods, native UI work, debugging, and shipping. The goal is not to collect isolated tricks, but to show how the different technical layers of STS2 modding fit together.

## Baseline Used Here

- STS2 release version: `v0.99.1`
- release date: `2026-03-13T20:40:28-07:00`
- engine line: `MegaDot v4.5.1.m.8.mono.custom_build`
- main pack Godot version: `4.5.1`

These version markers are included because STS2 modding advice is highly sensitive to the current Early Access build. A statement about manifests, install layout, hook shape, or localization merge paths is only useful if it is anchored to a known game version.

## Reading Order

1. [01-environment-setup.md](01-environment-setup.md)
2. [02-vanilla-sts2-layout.md](02-vanilla-sts2-layout.md)
3. [03-mod-loading-and-manifests.md](03-mod-loading-and-manifests.md)
4. [04-reverse-engineering-the-game.md](04-reverse-engineering-the-game.md)
5. [05-entrypoints-harmony-and-hooks.md](05-entrypoints-harmony-and-hooks.md)
6. [06-assets-pck-and-localization.md](06-assets-pck-and-localization.md)
7. [07-native-ui-and-runtime-inspection.md](07-native-ui-and-runtime-inspection.md)
8. [08-debugging-and-fast-iteration.md](08-debugging-and-fast-iteration.md)
9. [09-packaging-installing-and-verifying.md](09-packaging-installing-and-verifying.md)

Reference material:

- [index.md](index.md)
- [appendix-api-inventory.md](appendix-api-inventory.md)
- [appendix-example-mods.md](appendix-example-mods.md)
- [references.md](references.md)

If you want the shortest path through the repository, start with the numbered chapters. The appendices are there to support and verify the main narrative, not to replace it.

## License

This project is licensed under the [MIT license](LICENSE).
