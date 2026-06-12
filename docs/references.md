# References

## Current Release Evidence

Local files used for the `v0.103.3` baseline:

- `<STS2>/release_info.json`
- `<STS2>/data_sts2_windows_x86_64/sts2.runtimeconfig.json`
- `<STS2>/data_sts2_windows_x86_64/sts2.dll`
- `<STS2>/SlayTheSpire2.pck`
- current game and Godot user logs

Public release context:

- [Major Update #1 - v0.103.2](https://store.steampowered.com/news/app/2868840/view/526499614836130064)

The installed `v0.103.3` build is newer than that public major-update announcement. Local release metadata and assembly inspection are therefore used for exact current signatures.

## Toolchain

- [Godot 4.5.1 stable archive](https://godotengine.org/download/archive/4.5.1-stable/)
- [.NET 9](https://dotnet.microsoft.com/download/dotnet/9.0)
- [ILSpy](https://github.com/icsharpcode/ILSpy)
- [Harmony documentation](https://harmony.pardeike.net/articles/intro.html)
- [Harmony source](https://github.com/pardeike/Harmony)

The game installation currently also ships `tools/bin/godotpcktool.exe`. It is useful for selective PCK listing and extraction but is not a runtime dependency of released mods.

## Community Material

- [STS2 C# Modding Guide](https://lamali292.github.io/sts2_modding_guide/)
- [STS2 Early Access Mod Guide](https://www.reddit.com/r/slaythespire/comments/1rm5gvg/sts2_early_access_mod_guide/)

Community guides are orientation material. Revalidate manifests, signatures, install layout, and resource paths against the current game before copying code.
