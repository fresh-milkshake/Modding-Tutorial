# 07. Native UI And Runtime Inspection

## When This Matters

Most mods only need data, hooks, and packaging. This chapter matters when you want to reuse native STS2 UI screens or embed modded UI into the game's existing runtime hierarchy.

This is a distinct class of work. Once you move from gameplay rules into native UI reuse, the question stops being only which method should be patched and becomes which runtime branch owns geometry, layering, navigation, and tooltip behavior. STS2's UI is structured enough that those responsibilities are often separate.

## Relevant Native Types

For the current build, the most useful UI types are:

- `NOverlayStack`
- `NModalContainer`
- `NHoverTipSet`
- `NSubmenu`
- `NSubmenuStack`
- `NCardLibrary`
- `NCardLibraryGrid`
- `NRelicCollection`
- `NRelicCollectionEntry`

These types are useful not because every mod needs them, but because they reveal the architecture of the native UI. `NOverlayStack` and `NModalContainer` describe host layers. `NSubmenu` and `NSubmenuStack` describe lifecycle and navigation. `NCardLibrary` and `NRelicCollection` describe concrete reusable screens. Once those roles are separated, many UI regressions become easier to reason about.

## Visual Host vs Lifecycle Host

These are not the same concern.

Visual host candidates:

- `NOverlayStack`
- sometimes `NModalContainer`

Lifecycle and navigation owners:

- `NSubmenuStack`
- `NMainMenuSubmenuStack`

Safe rule:

- treat `NSubmenuStack` as lifecycle and back-navigation ownership
- do not assume it is the correct visual parent for submenu geometry

This distinction is the core idea of the chapter. A control can participate in one system for visibility and back-navigation while being rendered inside another system for coordinate space and layering. Mods that ignore this split often solve one problem by creating another: they fix navigation but break geometry, or fix visibility but break tooltips.

## Overlay And Tooltip Contract

For native picker-style integrations, the current safe baseline is:

- use a full-screen runtime UI branch that matches the game's own coordinate space
- keep tooltip hosts above the custom picker host
- keep custom dimming and input blocking below tooltip hosts
- keep native submenu screens stack-aware but not automatically parented under the stack

`NOverlayStack` is the primary visual host candidate for this style of work.

The reason is structural rather than stylistic. Tooltip systems, modal branches, and overlay branches all carry assumptions about sibling order and coordinate roots. If a mod inserts a screen into the wrong branch, the result may still render, but it will render under the wrong rules. That is why apparent layout bugs in STS2 UI work are often hierarchy bugs.

## Reusing Native Library Screens

`NCardLibrary` and `NRelicCollection` are reusable native screens, but they need the correct stack and host relationship.

Typical call sequence:

1. create the native screen
2. initialize it with the current run state if required
3. assign a submenu stack with `SetStack(...)`
4. attach the screen to a valid visual host
5. push the screen through the stack
6. show it

Safe rule:

- the screen should know its stack
- the stack should own lifecycle
- the screen should still live in the correct visual branch

This is a good example of STS2's division of responsibility. A screen does not become self-sufficient merely because it was created successfully. It still expects the surrounding runtime context to provide navigation semantics, visibility transitions, and an appropriate visual host.

## Common Failure Modes

These failures are worth documenting because they are easy to trigger:

- wrong host parent causes shifted geometry
- backdrop above tooltip hosts hides native hover tips
- treating `NSubmenuStack` as the screen's visual parent can make the screen disappear
- detached top-level overlays can break layering even when they appear to fix z-order

These are not unrelated bugs. They are different expressions of the same underlying rule: native UI components tend to assume a particular relationship between geometry, lifecycle, and layering. When a mod places a correct component into the wrong runtime context, the failure often appears one layer away from the actual mistake.

## Runtime Inspection Workflow

For UI bugs, use this order:

1. inspect the types and lifecycle in decompiled metadata
2. inspect the live runtime tree and parent chain
3. confirm sibling order around overlay, modal, and tooltip hosts
4. only then change hierarchy or layout code

Local debug-hook tooling can help here, but it is an optional research aid, not a tutorial prerequisite.

This order is intentionally conservative. UI debugging becomes expensive very quickly when nodes are moved around before it is clear which system currently owns them.
