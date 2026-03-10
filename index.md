# Slay the Spire 2 Modding Handbook

This folder is a repo-local handbook for modding **Slay the Spire 2 Early Access**. It is written for developers who already know basic C#, .NET, Godot, and decompilation workflows, and want the STS2-specific details without generic programming lessons.

## Scope

This handbook covers:

- environment setup
- the real on-disk structure of the installed game
- how STS2 discovers and initializes mods
- how to reverse-engineer the game safely
- how to structure entrypoints, Harmony patches, and hook-based integrations
- how assets, `.pck` payloads, and localization behave
- debugging and fast iteration
- packaging, installation, verification, and troubleshooting

This handbook does not try to teach C#, Godot, or Harmony from first principles.

## Source Hierarchy

The docs deliberately separate facts by source.

- **Verified from vanilla STS2 install:** game directory layout, `release_info.json`, `sts2.dll`, `SlayTheSpire2.pck`, log files.
- **Observed in real mods:** the six archives under [`EXAMPLE_MODS`](../EXAMPLE_MODS).
- **Community guide cross-check:** [lamali292's STS2 modding guide](https://lamali292.github.io/sts2_modding_guide/). This is the best public guide I found, but it is treated as a reference, not as the final source of truth.
- **Repo-local helper tooling:** the `tools/` folder inside this install. It is useful, but it is **not** part of vanilla STS2.

## Reading Order

1. [01-environment-setup.md](01-environment-setup.md)
2. [02-vanilla-sts2-layout.md](02-vanilla-sts2-layout.md)
3. [03-mod-loading-and-manifests.md](03-mod-loading-and-manifests.md)
4. [04-reverse-engineering-the-game.md](04-reverse-engineering-the-game.md)
5. [05-entrypoints-harmony-and-hooks.md](05-entrypoints-harmony-and-hooks.md)
6. [06-assets-pck-and-localization.md](06-assets-pck-and-localization.md)
7. [07-debugging-and-fast-iteration.md](07-debugging-and-fast-iteration.md)
8. [08-packaging-installing-and-verifying.md](08-packaging-installing-and-verifying.md)

Reference material:

- [appendix-api-inventory.md](appendix-api-inventory.md)
- [appendix-example-mods.md](appendix-example-mods.md)
- [references.md](references.md)

## Quick Start For Experienced Modders

If you already know Harmony and Godot:

1. Confirm your baseline in [01-environment-setup.md](01-environment-setup.md).
2. Read [03-mod-loading-and-manifests.md](03-mod-loading-and-manifests.md) to avoid packaging mistakes.
3. Read [05-entrypoints-harmony-and-hooks.md](05-entrypoints-harmony-and-hooks.md) for STS2-native entry patterns.
4. Read [06-assets-pck-and-localization.md](06-assets-pck-and-localization.md) before you ship assets or strings.
5. Use [08-packaging-installing-and-verifying.md](08-packaging-installing-and-verifying.md) as your release checklist.

## Baseline Used For This Handbook

These docs were verified against the locally installed game state:

- STS2 release version: `v0.98.2`
- release date: `2026-03-06T15:52:37-08:00`
- engine line seen in `sts2_stdout.log`: `MegaDot v4.5.1.m.8.mono.custom_build`
- pack metadata seen from `SlayTheSpire2.pck`: Godot `4.5.1`

If STS2 updates, re-check anything version-sensitive before assuming the same behavior still applies.
