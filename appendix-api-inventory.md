# Appendix A. API Inventory

This appendix is a compact inventory of STS2 modding-relevant surfaces verified from the current `sts2.dll`.

## `MegaCrit.Sts2.Core.Modding.ModManifest`

Observed fields:

- `author`
- `description`
- `name`
- `pckName`
- `version`

Practical meaning:

- this is the manifest shape the loader understands
- real mod JSON uses `pck_name` on disk to populate `pckName`

## `MegaCrit.Sts2.Core.Modding.ModManager`

Observed public members of interest:

- `Initialize()`
- `LoadModsInDirRecursive(Godot.DirAccess dirAccess, ModSource source)`
- `TryLoadModFromPck(string pckFilename, Godot.DirAccess dirAccess, ModSource source)`
- `TryLoadModFromSteam(PublishedFileId_t workshopItemId)`
- `CallModInitializer(Type initializerType)`
- `GetModdedLocTables()`
- `GetModNameList()`
- `AllMods`
- `LoadedMods`
- `PlayerAgreedToModLoading`

Interpretation:

- STS2 has a real mod manager, not just a loose assembly scanner
- it loads recursively from directories
- it handles `.pck` payloads explicitly
- it can source mods from local directories and Steam Workshop
- it has a dedicated path for modded localization tables

## `MegaCrit.Sts2.Core.Modding.ModInitializerAttribute`

Observed constructor:

- `.ctor(string initializerMethod)`

Observed real usages in example mods:

- `UpgradeAllCards.UpgradeAllCardsMod => ModInitializer("Initialize")`
- `BetterSpire2.ModEntry => ModInitializer("Init")`
- `DamageMeter.MainFile => ModInitializer("Initialize")`

Interpretation:

- the attribute is applied to a type
- the string names the initializer method STS2 should call

## `MegaCrit.Sts2.Core.Modding.Mod`

Observed fields:

- `modSource`
- `pckName`
- `wasLoaded`
- `manifest`
- `assembly`
- `assemblyLoadedSuccessfully`

Interpretation:

- STS2 tracks both manifest metadata and assembly loading state separately

## `MegaCrit.Sts2.Core.Modding.ModHelper`

Observed methods of interest:

- `AddModelToPool()`
- `AddModelToPool(Type poolType, Type modelType)`
- `ConcatModelsFromMods(...)`

Interpretation:

- there is explicit helper support for merging modded models into pool-style game content
- if you are adding cards, relics, or similar content, inspect this area before writing invasive patches

## `MegaCrit.Sts2.Core.Modding.ModSettings`

Observed fields and properties:

- `PlayerAgreedToModLoading`
- `DisabledMods`

Observed methods of interest:

- `IsModDisabled(Mod mod)`
- `IsModDisabled(string pckName, ModSource source)`

Interpretation:

- the game stores player consent and mod disablement state

## `MegaCrit.Sts2.Core.Modding.ModSource`

Observed enum values:

- `None = 0`
- `ModsDirectory = 1`
- `SteamWorkshop = 2`

## `MegaCrit.Sts2.Core.Hooks.Hook`

The hook class exposes a large semantic extension surface. Grouped categories include:

- combat lifecycle
- turn flow
- damage and block
- cards and piles
- map and rooms
- rewards and merchants
- potions, powers, healing, gold, stars
- death prevention and combat-ending checks
- modifier hooks for card rewards, maps, prices, damage, block, costs, draw, and more

Representative methods:

- `BeforeCombatStart`
- `AfterCombatEnd`
- `AfterPlayerTurnStart`
- `AfterDamageGiven`
- `AfterDamageReceived`
- `ModifyDamage`
- `ModifyBlock`
- `BeforeCardPlayed`
- `AfterCardPlayed`
- `AfterCardDrawn`
- `BeforeRoomEntered`
- `AfterRoomEntered`
- `ModifyRewards`
- `ShouldGainGold`
- `AfterGoldGained`
- `BeforePotionUsed`
- `AfterPotionUsed`
- `ShouldDie`
- `AfterPreventingDeath`

Representative signature samples confirmed locally:

- `Task AfterGoldGained(IRunState runState, Player player)`
- `bool ShouldGainGold(IRunState runState, CombatState combatState, decimal amount, Player player)`
- `Task AfterPlayerTurnStart(CombatState combatState, PlayerChoiceContext choiceContext, Player player)`
- `Task AfterRoomEntered(IRunState runState, AbstractRoom room)`

## Practical Use

If you are deciding where to integrate:

- start with `Hook` for semantically named gameplay events
- use Harmony when no hook matches the behavior you need
- inspect `ModHelper` when you are registering new models or extending pools
- use `ModManager` and `ModManifest` knowledge to debug load failures before touching gameplay code
