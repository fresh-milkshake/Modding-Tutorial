# References

## Current Release Evidence

Local files used for the `v0.107.1` baseline:

- `<STS2>/release_info.json`
- `<STS2>/data_sts2_windows_x86_64/sts2.runtimeconfig.json`
- `<STS2>/data_sts2_windows_x86_64/sts2.dll`
- `<STS2>/data_sts2_windows_x86_64/sts2.xml`
- `<STS2>/SlayTheSpire2.pck`
- current game and Godot user logs

Public release context:

- [Official STS2 announcement feed](https://steamcommunity.com/app/2868840/announcements/?l=english)

The installed `v0.107.1` release is the stable Major Update #2 baseline. Official announcements distinguish beta changes from the stable branch. API and resource inspection was repeated on 2026-11-04.

## Toolchain

- [Godot 4.5.1 stable archive](https://godotengine.org/download/archive/4.5.1-stable/)
- [.NET 9](https://dotnet.microsoft.com/download/dotnet/9.0)
- [ILSpy](https://github.com/icsharpcode/ILSpy)
- [Harmony documentation](https://harmony.pardeike.net/articles/intro.html)
- [Harmony source](https://github.com/pardeike/Harmony)

The inspected local installation contains `tools/bin/godotpcktool.exe`. This establishes local availability, not inclusion in every vanilla installation. It is an optional development tool and is not a runtime dependency of released mods.

## Verification Boundaries

The tutorial was audited against the installed `v0.107.1` assembly, its XML documentation, runtime configuration, and PCK. Loader behavior, hook ordering, model factories, UI stack methods, localization merging, and save-property serialization were traced in the decompiled implementation. Historical pack headers and contents were inspected separately.

Complete content and GUI class examples were compiled against the installed game DLLs. GUI source-generation output was also checked for callback bindings. Short fragments and placeholders are explanatory snippets rather than standalone projects.

The example mod has not been exercised in a running game during this audit. In-game save restoration, multiplayer behavior, texture rendering, focus, and tooltip order remain the explicit validation scenarios in the chapters, rather than reported test outcomes. Public release announcements provide branch context, not a substitute for inspecting the binary.

## Community Material

- [STS2 C# Modding Guide](https://lamali292.github.io/sts2_modding_guide/)
- [STS2 Early Access Mod Guide](https://www.reddit.com/r/slaythespire/comments/1rm5gvg/sts2_early_access_mod_guide/)

Community guides are orientation material. Revalidate manifests, signatures, install layout, and resource paths against the current game before copying code.
