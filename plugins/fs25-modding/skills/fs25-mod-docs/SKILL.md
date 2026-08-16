---
name: fs25-mod-docs
description: Use when finding FS25 modding documentation, API references, decompiled source code, or community resources. Helps locate the right reference material for scripting, GUI, vehicles, specializations, and other FS25 modding topics.
---

# FS25 Modding Documentation & References

## Overview

FS25 modding requires access to accurate API documentation, decompiled source code, and community-tested patterns. This skill maps the available documentation resources and provides a decision framework for finding the right reference for any modding question.

## When to Use

- Looking up FS25 API function signatures or parameters
- Understanding how a base game system works internally
- Finding implementation patterns for common modding tasks
- Debugging why a mod doesn't work as expected
- Setting up decompilation tools for game files
- Searching for community-tested solutions

## Resource Decision Framework

```
┌─────────────────────────────────────────────────────────────────────┐
│  YOUR QUESTION                           WHERE TO LOOK              │
├─────────────────────────────────────────────────────────────────────┤
│  "What params does loadGui() take?" →   Community LUADOC (API)      │
│  "How does Giants implement X?"     →   FS25-lua-scripting (source) │
│  "How do I build a dialog?"         →   AI Coding Reference (patterns)│
│  "Why doesn't this work?"           →   AI Coding Reference (pitfalls)│
│  "I need to decompile game files"   →   fs-utils (tools)            │
└─────────────────────────────────────────────────────────────────────┘
```

## Online Resources

### 1. FS25 Community LUADOC

**URL**: https://github.com/umbraprior/FS25-Community-LUADOC

The most comprehensive API reference available. Maintained by the community.

| Metric | Value |
|---|---|
| Documentation Pages | **1,661** |
| Script Functions | **11,102** |
| Coverage | Engine, Foundation, Script APIs |

**Structure:**

| Directory | Contains |
|---|---|
| `docs/engine/` | Engine-level APIs (I3D, Physics, Rendering, Sound, etc.) |
| `docs/foundation/` | Foundation APIs (Input, Scenegraph) |
| `docs/script/` | Script-level APIs (Vehicles, Specializations, GUI, Events, etc.) |

**Key subdirectories under `docs/script/`:**

| Directory | Contains |
|---|---|
| `script/Vehicles/` | Vehicle base classes (Vehicle, VehicleMotor, VehicleSystem) |
| `script/Specializations/` | All vehicle/placeable specializations (FillUnit, Trailer, Drivable, etc.) |
| `script/GUI/` | GUI framework (GuiElement, Overlay, dialogs) |
| `script/Events/` | Network event classes |
| `script/Economy/` | Economy, prices, selling points |
| `script/FillTypes/` | Fill type system |
| `script/Utils/` | Utility functions |
| `script/Base/` | Core base classes (BaseMission, FSBaseMission) |

**Quick Links:**
- [GUI](https://github.com/umbraprior/FS25-Community-LUADOC/tree/main/docs/script/GUI)
- [Vehicles](https://github.com/umbraprior/FS25-Community-LUADOC/tree/main/docs/script/Vehicles)
- [Specializations](https://github.com/umbraprior/FS25-Community-LUADOC/tree/main/docs/script/Specializations)
- [Events](https://github.com/umbraprior/FS25-Community-LUADOC/tree/main/docs/script/Events)
- [Economy](https://github.com/umbraprior/FS25-Community-LUADOC/tree/main/docs/script/Economy)
- [Engine](https://github.com/umbraprior/FS25-Community-LUADOC/tree/main/docs/engine)

**Usage**: Search for the specific `.md` file matching the class or function you need. Each file documents the class's methods, parameters, and return values.

### 2. FS25-lua-scripting (Raw Source Archive)

**URL**: https://github.com/Dukefarming/FS25-lua-scripting

Raw Lua source files from the game's dataS folder. **267 Lua files** including Vehicle.lua, VehicleMotor.lua, dialogs, and managers.

**Best for**: Understanding internal implementations — how Giants actually wrote the code.

**Note**: Archived (April 2025) but still valuable as a reference.

### 3. FS25 AI Coding Reference

**URL**: https://github.com/XelaNull/FS25_UsedPlus/tree/master/FS25_AI_Coding_Reference

Battle-tested implementation patterns and pitfalls. Built by analyzing 164+ community mods.

**Structure:**

| Directory | Contains | Status |
|---|---|---|
| `basics/` | modDesc.xml, localization, input bindings, Lua patterns | ✅ Validated |
| `patterns/` | GUI dialogs, events, managers, save/load, extensions | ✅ Validated |
| `advanced/` | Placeables, triggers, vehicles, HUD, animations | ⚠️ Partial |
| `pitfalls/` | 17 common mistakes and fixes | ✅ Battle-tested |

**Key patterns covered:**
- `patterns/gui-dialogs.md` — MessageDialog pattern, XML
- `patterns/events.md` — Network events for multiplayer sync
- `patterns/managers.md` — Singleton managers for global state
- `patterns/save-load.md` — Savegame persistence
- `patterns/extensions.md` — Hooking game classes
- `patterns/vehicle-info-box.md` — Custom HUD info
- `patterns/shop-ui.md` — Shop customization
- `patterns/message-center.md` — Event subscription
- `advanced/hud-framework.md` — Interactive HUD displays
- `advanced/vehicle-configs.md` — Equipment configurations

**Critical pitfalls to avoid:**
- `os.time()` → use `g_currentMission.time` (sandboxed Lua)
- `goto` / `::label::` → Lua 5.1 only, use `if not then`
- `Slider` widgets → use `MultiTextOption` (unreliable events)
- `DialogElement` → use `MessageDialog` (rendering issues)
- `g_gui:showYesNoDialog()` → use `YesNoDialog.show()` (doesn't exist)

### 4. fs-utils

**URL**: https://github.com/scfmod/fs-utils

A collection of command-line tools for decompiling and reading FS game files.

| Tool | Purpose |
|---|---|
| `fs-luau-decompile` | Decompile Luau `.l64` bytecode (FS25) |
| `fs-luajit-decompile` | Decompile LuaJIT `.l64` bytecode (FS19/FS22) |
| `fs-shapes-unlock` | Unlock `.i3d.shapes` files |
| `fs-unpack` | Extract `.gar`/`.dlc` archives |
| `fs-xml-format` | Format XML files |

**Usage examples:**
```bash
# Decompile a single FS25 script
fs-luau-decompile dataS.gar/scripts/main.l64

# Decompile a directory recursively
fs-luau-decompile -r dataS.gar/scripts/vehicles/ ./output/

# Extract a GAR archive
fs-unpack dataS.gar ./output/

# Unlock shapes files
fs-shapes-unlock -r ./models/ ./output/
```

### 5. Community Mods — Downloadable Examples

For real-world mod examples, download mods from the official Farming Simulator mod hub:

**URL**: https://www.farming-simulator.com/mods.php

**Usage**: Pick a mod that implements a feature similar to what you're building, download it, and study its structure. Mods are zip files — extract them to view the `modDesc.xml`, Lua scripts, GUI files, and translations. This is the best way to see how other modders structure their code for specific features.

---

## Decompiling Game Files

To access the game's actual source code, decompile the `.l64` bytecode files from the game's `dataS.gar` archive:

1. Clone and build `fs-utils` from https://github.com/scfmod/fs-utils
2. Build the needed tool: `cargo build --release -p fs-luau-decompile`
3. Locate the game's `dataS.gar` archive (in the game installation directory)
4. Decompile: `fs-luau-decompile -r dataS.gar/scripts/ ./output/`
5. Search the decompiled output for the class/function you need

**Decompiled GUI source** is also available from the game's `dataS.gar` — extract the `gui/` directory to access base GUI framework, elements, dialogs, and HUD implementations.

---

## Key Globals Reference

```lua
g_currentMission     -- Current game session
g_server             -- Server instance (nil on client)
g_client             -- Client instance
g_farmManager        -- Farm data access
g_storeManager       -- Shop/store items
g_vehicleTypeManager -- Vehicle type registry
g_gui                -- GUI system
g_i18n               -- Localization
g_messageCenter      -- Event pub/sub system
g_specializationManager -- Specialization registry
g_fillTypeManager    -- Fill type registry
g_currentModName     -- Current mod's name
g_currentModDirectory -- Current mod's directory
```

## Common MessageTypes

```lua
MessageType.HOUR_CHANGED    -- Every game hour
MessageType.DAY_CHANGED     -- Every game day
MessageType.PERIOD_CHANGED  -- Season change
MessageType.YEAR_CHANGED    -- New year
MessageType.MONEY_CHANGED   -- Farm money changes
```

---

## Workflow: Finding Documentation

1. **Identify what you need** — API signature, implementation detail, or pattern?
2. **Check for locally available references** — search any downloaded docs, decompiled source, or gamedata directories in the workspace
3. **If not found locally**, use the online resources:
   - API signature → Community LUADOC
   - Implementation detail → FS25-lua-scripting
   - Pattern/pitfall → AI Coding Reference
4. **For decompilation needs**, use fs-utils tools
5. **For GUI work**, decompile the game's `gui/` directory for base element implementations

---

## Quick Reference

| Question | Where to Look |
|---|---|
| API function signature? | Community LUADOC |
| How does Giants implement X? | FS25-lua-scripting or decompiled source |
| How do I build a dialog? | AI Coding Reference `patterns/gui-dialogs.md` |
| Why doesn't this work? | AI Coding Reference `pitfalls/what-doesnt-work.md` |
| GUI element API? | Decompiled game `gui/elements/` |
| HUD element API? | Decompiled game `gui/hud/` |
| Vehicle specialization API? | Community LUADOC `script/Specializations/` |
| Network event pattern? | AI Coding Reference `patterns/events.md` |
| Save/load pattern? | AI Coding Reference `patterns/save-load.md` |
| Need to decompile files? | fs-utils |
| Community mod example? | https://www.farming-simulator.com/mods.php |