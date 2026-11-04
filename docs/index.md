# Slay the Spire 2 Modding Tutorial

Version-aware documentation for building local C# and Godot content mods for **Slay the Spire 2 Early Access**.

## Coverage

- current game layout, loader behavior, and version checks
- `net9.0` projects referencing the assemblies shipped with the game
- the external mod manifest, dependencies, staging, and installation
- decompiling `sts2.dll` and inspecting `SlayTheSpire2.pck`
- `[ModInitializer]`, semantic hooks, and Harmony patches
- model discovery, pools, unlock visibility, and localization
- detailed custom card, potion, relic, act event, and GUI examples
- Godot `.pck` resources and native UI integration
- logs, save-state troubleshooting, packaging, and release verification

## Verified Baseline

| Item | Value |
| --- | --- |
| STS2 | `v0.107.1` |
| Commit | `59260271` |
| Release metadata date | `2026-06-18T15:43:56-07:00` |
| Target framework | `net9.0` |
| Bundled .NET runtime | `9.0.7` |
| Main PCK format | Godot `4.5.1` |

Early Access updates can change signatures and loader contracts. Re-check `release_info.json`, `sts2.runtimeconfig.json`, and `sts2.dll` before maintaining a released mod.

## Start Reading

Begin with [Environment Setup](01-environment-setup.md), then follow the numbered chapters in the sidebar. The first nine chapters explain the runtime and packaging model. Chapters 10–14 apply it to a card, potion, relic, act event, and custom overlay. [Chapter 15](15-state-saves-and-randomness.md) develops persistent state, cloning, and deterministic randomness through a counter-based relic.

Use the [Handbook Overview](handbook-overview.md) for the complete structure and source hierarchy.

## Not Covered

This tutorial does not teach C#, Harmony, Godot, illustration, or game design from first principles. It documents how those tools connect to the current STS2 runtime.

## Verification And Branches

Last API and resource inspection: **2026-11-04**, stable `v0.107.1` (`59260271`). This baseline describes the installed stable branch. The public beta is a separate compatibility target; do not assume its APIs or balance changes are present here.

The code examples are checked by compilation against the installed assemblies. Compilation and resource inspection do not replace the in-game validation matrices in each chapter.
