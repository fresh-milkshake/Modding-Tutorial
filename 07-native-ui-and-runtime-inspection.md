# 07. Native UI And Runtime Inspection

## When This Matters

Most mods only need data, hooks, and packaging. This chapter matters when you want to reuse native STS2 UI screens or embed modded UI into the game's existing runtime hierarchy.

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

## Overlay And Tooltip Contract

For native picker-style integrations, the current safe baseline is:

- use a full-screen runtime UI branch that matches the game's own coordinate space
- keep tooltip hosts above the custom picker host
- keep custom dimming and input blocking below tooltip hosts
- keep native submenu screens stack-aware but not automatically parented under the stack

`NOverlayStack` is the primary visual host candidate for this style of work.

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

## Common Failure Modes

These failures are worth documenting because they are easy to trigger:

- wrong host parent causes shifted geometry
- backdrop above tooltip hosts hides native hover tips
- treating `NSubmenuStack` as the screen's visual parent can make the screen disappear
- detached top-level overlays can break layering even when they appear to fix z-order

## Runtime Inspection Workflow

For UI bugs, use this order:

1. inspect the types and lifecycle in decompiled metadata
2. inspect the live runtime tree and parent chain
3. confirm sibling order around overlay, modal, and tooltip hosts
4. only then change hierarchy or layout code

Local debug-hook tooling can help here, but it is an optional research aid, not a tutorial prerequisite.
