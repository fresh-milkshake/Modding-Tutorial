# Appendix A. API Inventory

Verified against `sts2.dll` from STS2 `v0.107.1`, commit `59260271`.

## `ModManifest`

JSON fields:

- `id`
- `name`
- `author`
- `description`
- `version`
- `has_pck`
- `has_dll`
- `dependencies`
- `affects_gameplay`
- `min_game_version`

`id` is required. The loader resolves sibling `<id>.dll` and `<id>.pck` files from the manifest directory. `pck_name` is not a current field.

## `ModManager`

Public members of interest:

- `IReadOnlyList<Mod> Mods`
- `bool PlayerAgreedToModLoading`
- `event Action<Mod> OnModDetected`
- `event MetricsUploadHook OnMetricsUpload`
- `Task Initialize(IModManagerFileIo, ModSettings?, SemanticVersion?)`
- `ModManagerState State`
- `EvaluateBranchSupport(PlatformBranch, IReadOnlyList<(string MinBranch, string MaxBranch)>)`
- `GetModdedLocTables(string language, string file)`
- `GetGameplayRelevantModNameList()`
- `GetNonGameplayRelevantModNameList()`
- `IsRunningModded()`
- `HasHarmonyPatches()`
- `GetLoadedMods()`
- `Dispose()`

Directory scanning, manifest deserialization, dependency ordering, payload loading, and initializer invocation are internal implementation details. Inspect them for compatibility, but do not call them as a mod API.

## `Mod`

Current fields:

- `modSource`
- `path`
- `state`
- `manifest`
- `version` (parsed `SemanticVersion?`)
- `assembly`
- `errors`

`path` is the directory containing the external manifest.

## `ModInitializerAttribute`

Constructor:

```text
ModInitializerAttribute(string initializerMethod)
```

The named method must be static. If an assembly has no attributed type, the loader automatically calls Harmony `PatchAll` with an ID derived from author and mod ID.

## `ModHelper`

Current methods:

- `AddModelToPool<TPoolType, TModelType>()`
- `AddModelToPool(Type poolType, Type modelType)`
- `ConcatModelsFromMods<TModelType>(IPoolModel, IEnumerable<TModelType>)`
- `SubscribeForRunStateHooks(string, RunHookSubscriptionDelegate)`
- `SubscribeForCombatStateHooks(string, CombatHookSubscriptionDelegate)`
- `IterateAllRunStateSubscribers(RunState)`
- `IterateAllCombatStateSubscribers(CombatState)`

Pool additions must occur before the pool is first materialized. Hook-subscriber IDs must be unique.

## `ModelDb`

Relevant behavior:

- discovers concrete mod `AbstractModel` subtypes through loaded assemblies
- constructs canonical instances with public parameterless constructors
- derives IDs from model category and class name
- exposes typed helpers such as `Card<T>`, `Potion<T>`, `Relic<T>`, and `Event<T>`
- aggregates pools, acts, events, encounters, and other model families

Discovery is not pool membership. Cards, potions, and relics still require `ModHelper.AddModelToPool`.

## Base Models

### `CardModel`

Current constructor:

```text
CardModel(int canonicalEnergyCost, CardType, CardRarity, TargetType, bool shouldShowInCardLibrary = true)
```

Important overrides and properties:

- `CanonicalVars`
- `OnPlay(PlayerChoiceContext, CardPlay)`
- `OnUpgrade()`
- `PortraitPath`
- `Pool`
- `CanonicalKeywords`
- `CanonicalTags`
- `GetResultPileTypeForCardPlay()`

Public description method:

```text
GetDescriptionForPile(PileType, Creature? target = null)
```

### `PotionModel`

Required properties:

- `Rarity`
- `Usage`
- `TargetType`

Important members:

- `CanonicalVars`
- `OnUse(PlayerChoiceContext, Creature?)`
- `AssertValidForTargetedPotion`
- `CanBeGeneratedInCombat`
- custom usability checks
- `ExtraHoverTips`

`Pool` requires membership in a current potion pool. Image lookup uses fixed atlas-resource paths derived from the model ID.

### `RelicModel`

Required property:

- `Rarity`

Important members:

- `CanonicalVars`
- semantic hook overrides inherited from `AbstractModel`
- `PackedIconPath`
- `PackedIconOutlinePath`
- `BigIconPath`
- `AfterObtained()`
- `AfterRemoved()`
- `IsAllowed(IRunState)`

`Pool` requires membership in a current relic pool.

### `EventModel`

Important members:

- `GenerateInitialOptions()`
- `InitialDescription`
- `IsAllowed(IRunState)`
- `CalculateVars()`
- `SetEventState(...)`
- `SetEventFinished(...)`
- `LayoutType`
- `GetAssetPaths(IRunState)`
- event combat and resume helpers

Events are discovered by `ModelDb` but are selected through concrete act event lists or `ModelDb.AllSharedEvents`.

## UI

Current integration surfaces:

- `NOverlayStack.Instance`
- `NOverlayStack.Push(IOverlayScreen)`
- `NOverlayStack.Remove(IOverlayScreen)`
- `NModalContainer.Instance`
- `NSubmenuStack`
- `NCardLibrary.Create()`
- `NRelicCollection.Create()`
- `ActiveScreenContext`

`NOverlayStack` owns overlay visuals and lifecycle. `NSubmenuStack.Push` manages navigation without attaching the screen; concrete stack factories create and parent supported screens. The current run stack parents its cached libraries under itself.

## Hooks

`Hook.AfterCardPlayed` currently invokes model listeners in two passes:

1. `AfterCardPlayed(PlayerChoiceContext, CardPlay)`
2. `AfterCardPlayedLate(PlayerChoiceContext, CardPlay)`

The late pass follows ordinary post-play listeners for each individual play. Both passes precede final result-pile movement and are skipped if combat has ended. See [card resolution](05-entrypoints-harmony-and-hooks.md#card-resolution).

## Version And Loader Types

`ModDependency` contains JSON fields `id` and optional `min_version`; `ModManifest.dependencies` is `List<ModDependency>?`. `ModManifest.ReadFromStream(Stream, out List<LocString>?)` reads manifests and migrates legacy string dependencies with a diagnostic.

`ModManagerState` values are `None`, `Initialized`, and `Skipped`. This is loader initialization state, separate from each `Mod.state`. `Skipped` covers test/AOT paths and the `nomods` command-line option. Per-mod state, errors, and parsed version should be inspected when diagnosing compatibility.

See [versioned manifests](03-mod-loading-and-manifests.md#versioned-dependencies) for the authoring contract.
