# 14. Custom GUI

This chapter builds an asset-free run overlay opened with `F8`. It demonstrates the minimum native integration contract: attach a controller when `NRun` is ready, implement `IOverlayScreen`, and let `NOverlayStack` own focus, dimming, and overlay order.

The example intentionally creates controls in C#. Production interfaces can move the same hierarchy into a `.tscn` scene after the lifecycle is proven.

## Attach After `NRun` Exists

The mod initializer runs before a run UI exists. Do not create Godot nodes there. Patch `NRun._Ready` and attach one controller to that run node:

```csharp
using HarmonyLib;
using MegaCrit.Sts2.Core.Nodes;

namespace FieldNotes.Gui;

[HarmonyPatch(typeof(NRun), nameof(NRun._Ready))]
internal static class NRunReadyPatch
{
    private const string ControllerName = "FieldNotesGuiController";

    private static void Postfix(NRun __instance)
    {
        if (__instance.GetNodeOrNull(ControllerName) is not null)
        {
            return;
        }

        __instance.AddChild(new ResearchGuiController
        {
            Name = ControllerName
        });
    }
}
```

The patch is re-entered for every new `NRun`, so duplicate protection is local to the current scene tree rather than stored in a static flag.

## Input Controller

```csharp
using Godot;

namespace FieldNotes.Gui;

public sealed partial class ResearchGuiController : Node
{
    public override void _UnhandledKeyInput(InputEvent @event)
    {
        if (@event is not InputEventKey
            {
                Pressed: true,
                Echo: false,
                Keycode: Key.F8
            })
        {
            return;
        }

        ResearchOverlay.Open();
        GetViewport().SetInputAsHandled();
    }
}
```

Use `_UnhandledKeyInput` so focused native controls receive their normal input first. A released mod should expose configurable input rather than permanently claiming a hardcoded key.

## Overlay Implementation

```csharp
using System.Linq;
using Godot;
using MegaCrit.Sts2.Core.Entities.Multiplayer;
using MegaCrit.Sts2.Core.Models;
using MegaCrit.Sts2.Core.Nodes.Screens.Overlays;

namespace FieldNotes.Gui;

public sealed partial class ResearchOverlay : Control, IOverlayScreen
{
    private Button? _closeButton;

    public NetScreenType ScreenType => NetScreenType.None;

    public bool UseSharedBackstop => true;

    public Control? DefaultFocusedControl => _closeButton;

    public static void Open()
    {
        var stack = NOverlayStack.Instance;
        if (stack is null || stack.Peek() is ResearchOverlay)
        {
            return;
        }

        stack.Push(new ResearchOverlay
        {
            Name = "FieldNotesResearchOverlay"
        });
    }

    public override void _Ready()
    {
        SetAnchorsAndOffsetsPreset(LayoutPreset.FullRect);
        MouseFilter = MouseFilterEnum.Stop;

        var center = new CenterContainer();
        center.SetAnchorsAndOffsetsPreset(LayoutPreset.FullRect);
        AddChild(center);

        var panel = new PanelContainer
        {
            CustomMinimumSize = new Vector2(860f, 520f)
        };
        center.AddChild(panel);

        var margin = new MarginContainer();
        margin.AddThemeConstantOverride("margin_left", 32);
        margin.AddThemeConstantOverride("margin_top", 28);
        margin.AddThemeConstantOverride("margin_right", 32);
        margin.AddThemeConstantOverride("margin_bottom", 28);
        panel.AddChild(margin);

        var column = new VBoxContainer();
        column.AddThemeConstantOverride("separation", 20);
        margin.AddChild(column);

        column.AddChild(new Label
        {
            Text = "Field Research",
            HorizontalAlignment = HorizontalAlignment.Center
        });

        column.AddChild(new RichTextLabel
        {
            BbcodeEnabled = true,
            FitContent = true,
            CustomMinimumSize = new Vector2(760f, 350f),
            Text = BuildSummary()
        });

        _closeButton = new Button
        {
            Text = "Close"
        };
        _closeButton.Pressed += Close;
        column.AddChild(_closeButton);
    }

    public void AfterOverlayOpened()
    {
    }

    public void AfterOverlayClosed()
    {
        QueueFree();
    }

    public void AfterOverlayShown()
    {
        Visible = true;
        _closeButton?.GrabFocus();
    }

    public void AfterOverlayHidden()
    {
        Visible = false;
    }

    private static string BuildSummary()
    {
        return $"[b]Loaded model database[/b]\n\n" +
               $"Cards: {ModelDb.AllCards.Count()}\n" +
               $"Potions: {ModelDb.AllPotions.Count()}\n" +
               $"Relics: {ModelDb.AllRelics.Count()}\n" +
               $"Events: {ModelDb.AllEvents.Count()}\n\n" +
               "This panel is hosted by NOverlayStack and uses its shared backstop.";
    }

    private void Close()
    {
        var stack = NOverlayStack.Instance;
        if (stack is null)
        {
            QueueFree();
            return;
        }

        stack.Remove(this);
    }
}
```

## Lifecycle Meaning

`NOverlayStack.Push` performs more than `AddChild`:

- hides the previous overlay
- adds the new control to the overlay branch
- places the shared backstop relative to the overlay
- updates `ActiveScreenContext`
- calls open and shown callbacks

`Remove` calls hidden and closed callbacks, reveals the previous overlay, and updates focus context. It does not guarantee that an arbitrary custom control frees itself, so the example calls `QueueFree` in `AfterOverlayClosed`.

`Visible` is changed in shown and hidden callbacks because stacked overlays remain in the tree while another overlay is above them.

## Focus And Input

`DefaultFocusedControl` lets the active screen context identify the close button. The shown callback also calls `GrabFocus` so keyboard and controller navigation begin in a usable state.

For a larger panel:

- define explicit focus neighbors for grids
- provide a cancel action that calls `Close`
- preserve the previously focused native control if returning focus is important
- do not process gameplay hotkeys while a text field or modal is active

The `NetScreenType.None` value means the example does not advertise itself as one of the game's synchronized network screens. A multiplayer UI that changes shared game state needs a separate network and command design; changing the enum does not create synchronization.

## Native Widgets Versus Plain Godot Controls

Plain `Button`, `Label`, and containers are sufficient to demonstrate hierarchy, but they do not automatically match STS2 typography, sounds, hover motion, or controller behavior.

For native presentation:

1. inspect a nearby native screen in `sts2.dll`
2. inspect its `.tscn` and theme resources in the PCK
3. instantiate native button or label scenes rather than copying private fields
4. preserve the overlay lifecycle shown here

Do not start by cloning an entire complex native screen. First make a small overlay open, focus, hide, close, and free correctly.

## Reusing Card And Relic Screens

`NCardLibrary` and `NRelicCollection` are `NSubmenu` screens, not `IOverlayScreen` implementations. They need submenu-stack ownership and the visual geometry used by their native caller.

If a custom overlay launches one of them:

- keep the overlay as the visual and input context that launched the flow
- create the native screen through its factory
- assign the correct submenu stack
- attach it to the visual host used by the native compendium path
- let the submenu stack push and pop it
- keep tooltip layers above custom dimming controls

Attaching a native submenu directly under `NSubmenuStack` can give it lifecycle but an invalid coordinate root. Attaching it only under `NOverlayStack` can give it geometry but no submenu navigation.

## Failure Modes

### Overlay never opens

Check that `NRun._Ready` was patched, the controller exists under the current run, and `NOverlayStack.Instance` is non-null.

### Overlay opens twice

Check both controller duplication and `stack.Peek()` protection. Static singleton flags often survive scene replacement and are less reliable than inspecting the current tree.

### Tooltips are hidden

The custom backdrop or panel is above the native tooltip layer. Compare sibling order with a vanilla overlay and move only the custom host, not the tooltip system.

### Geometry is shifted or clipped

The control is under a content-sized parent rather than the viewport-sized overlay branch. Log parent size, global position, and anchors.

### Close leaves an invisible blocker

The overlay was removed visually but not from the stack, or it was removed from the stack but not freed. Use `NOverlayStack.Remove` and free in `AfterOverlayClosed`.

## Validation Matrix

Verify:

1. one controller is attached for each run scene
2. `F8` opens only one overlay
3. the shared backstop blocks room input
4. the close button receives keyboard and controller focus
5. another native overlay can cover and reveal the custom overlay
6. close removes the screen from the stack and frees the node
7. map, rewards, hover tips, and pause screens still layer correctly
8. starting a new run creates a fresh controller without stale static state

Once this lifecycle is stable, replace the generated controls with a packed scene or expand the panel into a model browser, settings page, picker, or debugging surface.
