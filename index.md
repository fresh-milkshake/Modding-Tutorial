# Slay the Spire 2 Modding Handbook

This is the repo-local table of contents for the tutorial. The public landing page is [README.md](README.md).

## Scope

The handbook is limited to STS2-specific facts:

- local setup and version baseline
- game layout and built-in modding surface
- mod bootstrap, manifests, and packaging shape
- reverse engineering workflow
- behavior mods, content mods, and native UI reuse
- debugging, installation, and release validation

It does not try to teach C#, Godot, or Harmony from zero.

## Source Hierarchy

- **Vanilla install facts:** `release_info.json`, `sts2.dll`, `SlayTheSpire2.pck`, and game logs.
- **Observed mod patterns:** local working mods and unpacked release payloads.
- **Community cross-check:** [lamali292's guide](https://lamali292.github.io/sts2_modding_guide/).
- **Repo-local tooling:** useful helpers under `tools/`, but not part of vanilla STS2.

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

## Quick Start

If you already know Harmony and Godot:

1. confirm the baseline in [01-environment-setup.md](01-environment-setup.md)
2. read [03-mod-loading-and-manifests.md](03-mod-loading-and-manifests.md)
3. read [05-entrypoints-harmony-and-hooks.md](05-entrypoints-harmony-and-hooks.md)
4. read [06-assets-pck-and-localization.md](06-assets-pck-and-localization.md)
5. use [09-packaging-installing-and-verifying.md](09-packaging-installing-and-verifying.md) as the shipping checklist

## Baseline Used Here

- STS2 release version: `v0.99.1`
- release date: `2026-03-13T20:40:28-07:00`
- engine line: `MegaDot v4.5.1.m.8.mono.custom_build`
- main pack Godot version: `4.5.1`
- installed-mod UI verification date: `2026-03-23`

Version-sensitive note:

- archive layout and installed layout are not the same concern
- on this machine, the installed-mod UI reliably recognized root-level `mods\<ModId>.json`, `mods\<ModId>.dll`, and optional `mods\<ModId>.pck`
- folder-only installed payloads were not the safest recommendation for this local `v0.99.1` build

Re-check anything version-sensitive after a game update.
