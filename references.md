# References

## Primary Community Guide

- [lamali292 STS2 C# Modding Guide](https://lamali292.github.io/sts2_modding_guide/)
  - Best public STS2-focused walkthrough currently found.
  - Use it as the main community baseline, but verify version-sensitive claims against the live game.

## Supplementary Community Reference

- [Reddit: STS2 Early Access Mod Guide](https://www.reddit.com/r/slaythespire/comments/1rm5gvg/sts2_early_access_mod_guide/)
  - A compact community post covering the basics of local mod loading, `[ModInitializer]`, `.dll + .pck`, and remote debugging.

## Core Toolchain

- [Godot 4.5.1 stable archive](https://godotengine.org/download/archive/4.5.1-stable/)
  - Use the **.NET** build to match the game's current Godot line.
- [.NET 9 official download](https://dotnet.microsoft.com/download/dotnet/9.0)
  - Needed to build against the current STS2 managed runtime surface.
- [Visual Studio](https://visualstudio.microsoft.com/)
  - One good option for browsing assemblies and building mods.
- [ILSpy](https://github.com/icsharpcode/ILSpy)
  - Decompiler used to inspect `sts2.dll` and mod assemblies.
- [Harmony documentation](https://harmony.pardeike.net/articles/intro.html)
  - Reference for patch prefixes, postfixes, transpilers, and patch targeting.
- [Harmony GitHub repository](https://github.com/pardeike/Harmony)
  - Source and release home for Harmony.

## Vanilla STS2 Evidence Used In This Handbook

These are local files, not public web links:

- `release_info.json`
- `data_sts2_windows_x86_64\sts2.dll`
- `SlayTheSpire2.pck`
- `sts2_stdout.log`

Use them before trusting any tutorial that makes a version-sensitive claim.

## Repo-Local Helper Tooling

These are useful in this repository, but they are **not vanilla STS2 defaults**:

- `tools/bin/godotpcktool.exe`
- `tools/ilspy/`
- `tools/sts2-debug-hook/`
- `tools/sts2-loose-mod/`

Treat them as optional productivity tooling and internal research aids.
