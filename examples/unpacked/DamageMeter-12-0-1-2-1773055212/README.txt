DAMAGE METER - 0.1.2

Damage Meter is a combat stats and review HUD for Slay the Spire 2.
It tracks combat output, actual damage, resource use, and per-fight or run-wide performance so players can review what really happened in a run.

:: REQUIREMENTS ::
- Slay the Spire 2 with mod loading enabled
- Damage Meter installed correctly

:: FEATURES ::
- Tracks total output, actual damage, damage taken, block gained, and overkill
- Tracks cards played, draw/discard/exhaust flow, potions used, and debuffs applied
- Tracks energy spent and wasted energy
- Supports Current Fight and Total views
- Includes per-turn trend charts for team output, actual damage, and wasted energy
- Includes a dashboard view for quick category overview and player drill-down
- Stores lifetime records such as highest hit, best turn output, most fight output, most cards played, and most block gained
- HUD position, scale, opacity, max rows, and auto-clear behavior are configurable
- Includes English and Simplified Chinese localization that follows the game's language setting
- Works for both solo runs and multiplayer review use cases

:: QUICK START ::
1. Start a run and enter combat.
2. Press F7 to show or hide the HUD.
3. Use the left and right arrow buttons on the HUD header to switch categories.
4. Click the title to open the category list, or right-click the title / panel for the quick menu.
5. Click a player row to open that player's detail breakdown.
6. Click the segment label to switch between Current Fight, Total, and archived fight segments.
7. Click Overview Panel to open the dashboard view, then press Esc to close it.

:: CONTROLS ::
- F7 toggles the HUD
- Esc closes the dashboard view
- Left click and drag on the HUD panel moves it
- Right click on the HUD panel opens the category menu
- Settings lets you change scale, opacity, max rows, auto-clear, and reset HUD position
- Clear Stats resets saved Damage Meter records

:: INSTALLATION ::
1. Close Slay the Spire 2 before installing or updating the mod.
2. Extract the zip.
3. Copy the included `mods\DamageMeter` folder into your Slay the Spire 2 game directory.
4. If prompted, allow the existing DamageMeter files to be overwritten.
5. Launch the game and verify that Damage Meter appears in your mod list or works in combat.

:: SAVE AND SETTINGS MIGRATION ::
Damage Meter does not modify your run save files.
Its settings and lifetime records are stored separately in:
`%APPDATA%\Godot\app_userdata\DamageMeter\DamageMeter_settings.json`

For normal updates from an older version:
1. Close the game.
2. Back up your existing `mods\DamageMeter` folder if you want a rollback option.
3. Optionally back up `DamageMeter_settings.json` if you want to preserve HUD position and records.
4. Overwrite the old mod files with the new version.
5. Keep the settings JSON in place. Version 0.1.2 reads the same settings file and does not require manual conversion.

For migration to another PC or a clean reinstall:
1. Install the new version of the mod first.
2. Copy `DamageMeter_settings.json` from the old PC to the same path on the new PC.
3. If the `DamageMeter` folder does not exist yet under `app_userdata`, launch the game once and close it, then copy the file.
4. Launch the game again. Your HUD position, opacity, row limit, auto-clear setting, and lifetime records should be restored.

If you want a fully clean reset:
1. Close the game.
2. Delete `DamageMeter_settings.json`.
3. Launch the game again to let the mod create a fresh settings file.

:: NOTES ::
- This mod is useful both during a fight and after a fight when comparing team pace, resource usage, and burst turns.
- The HUD only updates when combat data exists, so some categories may stay empty outside of combat.

:: KNOWN ISSUES ::
- This is still an early public build and category coverage will expand over time
- Full run-to-run analytics are still limited in this version

:: CREDITS ::
- Mega Crit for Slay the Spire 2
- Harmony and the Godot modding toolchain
