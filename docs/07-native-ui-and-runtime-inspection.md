# 07. Native UI And Runtime Inspection

STS2 UI integration is primarily a hierarchy problem. A screen can have a lifecycle owner, a visual parent, a focus context, and a tooltip host that are related but not identical.

## Current Native Roles

Important types in `v0.107.1` include:

- `NOverlayStack`: run-time overlay host and stack
- `NModalContainer`: single modal host with a dimming backstop
- `NSubmenuStack`: submenu lifecycle and back-navigation owner
- `NSubmenu`: base class for native submenu screens
- `NCardLibrary`: reusable native card library screen
- `NRelicCollection`: reusable native relic collection screen
- `NHoverTipSet`: tooltip presentation layer
- `ActiveScreenContext`: focus ownership for the active screen

The names describe responsibilities, not interchangeable containers.

## Overlay Contract

`NOverlayStack.Instance` resolves from the active `NRun`. Its public contract is based on `IOverlayScreen`:

```csharp
public interface IOverlayScreen : IScreenContext
{
    NetScreenType ScreenType { get; }
    bool UseSharedBackstop { get; }
    void AfterOverlayOpened();
    void AfterOverlayClosed();
    void AfterOverlayShown();
    void AfterOverlayHidden();
}
```

`Push` adds the screen to the visual tree, hides the previous overlay, moves the shared backstop, updates active focus context, and emits a stack change signal. `Remove` reverses that lifecycle but does not itself free arbitrary custom nodes. A custom overlay should release itself in `AfterOverlayClosed`.

## Visual Host And Lifecycle Owner

`NSubmenuStack.Push` manages navigation and assigns stack ownership; it does not itself attach a screen to the scene tree. The concrete `NRunSubmenuStack.GetSubmenuType<T>()` factories create and cache supported screens, including `NCardLibrary` and `NRelicCollection`, and add them as children of that stack.

This explains a common failure pattern:

- calling `Push` on a newly created screen without attaching it leaves it outside the visual tree
- manually reparenting a native screen can change its coordinate root and clipping
- attaching a native submenu without its stack protocol omits native navigation ownership

Prefer the concrete stack's `GetSubmenuType<T>()` or `PushSubmenuType<T>()` for screens it supports. For a custom host, inspect and reproduce both the native attachment and navigation protocol. Parenting under a stack is not inherently invalid; the host's actual geometry determines the result.

## Reusing Native Libraries

`NCardLibrary` and `NRelicCollection` are concrete native screens with their own scene geometry. Their `Create` methods instantiate scenes from the game PCK. They expect submenu lifecycle, focus handling, preload state, and the same visual coordinate system used by the native compendium.

Before reusing one:

1. inspect its `Create`, `_Ready`, open, shown, hidden, and close methods
2. inspect where the vanilla caller attaches it
3. inspect which `NSubmenuStack` receives it
4. inspect asset preloading performed before creation
5. verify tooltip hosts remain above the screen and its dimming layer

Do not copy only the `Create()` call. Native scene creation is usually one step in a larger protocol.

## Tooltip And Layer Ordering

Tooltips often render in a sibling layer rather than inside the hovered control. A custom full-screen backdrop placed above that sibling can make hover logic run while hiding the actual tooltip.

The safe ordering is:

1. native room or menu content
2. custom dimming and input backstop
3. custom screen content
4. native tooltip and cursor layers

Exact node names can change. Verify the live tree instead of hardcoding an assumed sibling index.

## Coordinate Roots

Controls using full-rect anchors inherit the rectangle of their visual parent. Moving a screen between a viewport-sized host and a content-sized submenu container can shift or clip it even when all child anchors are correct.

When geometry is wrong, log:

- parent path
- global position
- size
- anchor values
- effective viewport size
- canvas layer and sibling index

Change the hierarchy only after these values identify the incorrect coordinate root.

## Runtime Inspection Workflow

1. decompile the screen and its native caller
2. inspect the packed scene path
3. open the relevant vanilla screen in game
4. inspect the live parent chain and sibling order
5. compare focus and tooltip behavior
6. reproduce the smallest valid lifecycle in the mod

Chapter 14 applies this model to a self-contained `IOverlayScreen` that does not depend on a custom `.tscn` file.

## Related Sections

Continue with the [complete custom GUI example](14-custom-gui.md#overlay-implementation), or the [event layout discussion](13-custom-act-events.md#portrait) for UI owned by an event.
