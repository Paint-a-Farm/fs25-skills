---
name: fs25-mod-structure
description: Use when structuring FS25 mod projects, organizing Lua script files, setting up modular architecture, or implementing standardized logging.
---

# FS25 Mod Structure & Organization

## Overview

A well-organized FS25 mod follows a consistent, proven structure used by the most popular community mods (UniversalAutoload, AutoDrive, TipAnywhere, WeatherForecastHUD, GarageMenu, etc.). This skill defines the standardized project hierarchy, file naming conventions, module architecture, and registration patterns that apply to **any** FS25 mod — vehicle mods, map mods, placeables, or pure script mods.

The goal is complete modularity: each Lua file has a single, well-defined responsibility, and the project structure makes it obvious where new code belongs.

## When to Use

- Setting up a new FS25 mod project structure
- Refactoring an existing mod for better organization
- Adding new systems, events, settings, or GUI elements to a mod
- Deciding where a new Lua file should live
- Understanding how to register specializations and inject into vehicle types

## Standardized Project Hierarchy

```
<ModName>/
├── .gitattributes              # Git line-ending normalization
├── artwork/                    # Mod icons, promotional images, banners (optional)
├── src/                        # All mod content (or at mod root for simple mods)
│   ├── modDesc.xml             # Mod descriptor (metadata, script entry points)
│   ├── modIcon.dds             # In-game mod icon
│   ├── register.lua            # Entry point: loads all scripts, registers specializations
│   ├── language/               # Translation files (l10n_en.xml, l10n_de.xml, etc.)
│   │   └── l10n_en.xml
│   ├── scripts/                # All Lua script files
│   │   ├── events/             # Network event classes
│   │   ├── gui/                # GUI / HUD / menu code (Lua + XML)
│   │   ├── utils/              # Shared utility modules
│   │   └── ...                 # Feature-specific modules
│   ├── gui/                    # GUI XML files (alternative location)
│   ├── textures/               # Texture files (DDS)
│   ├── sounds/                 # Audio files (OGG)
│   ├── drawing/                # I3D models, shaders, drawing assets
│   ├── triggers/               # Trigger I3D files
│   └── xml/                    # XML configuration data
```

### Key Principles

1. **Entry point at root**: `register.lua` (or `main.lua`) sits at the mod root and uses `source()` to load all dependencies in order.
2. **Scripts in `scripts/`**: All Lua files live under `scripts/`, organized into subdirectories by function.
3. **Events in `scripts/events/`**: Network event classes go in a dedicated `events/` subdirectory.
4. **Translations in `language/`**: Translation XML files use the `l10n_<lang>.xml` naming convention.
5. **GUI in `gui/`**: GUI Lua + XML files live together in a `gui/` directory.
6. **modDesc.xml lists entry points only**: `<extraSourceFiles>` contains the root entry point(s), not every individual script — the entry point loads the rest via `source()`.

---

## Directory Reference

### Root-Level Files

| File | Purpose |
|---|---|
| `modDesc.xml` | Mod descriptor — metadata, entry points, actions, input bindings |
| `register.lua` | Entry point — loads all scripts via `source()`, registers specializations, hooks game managers |
| `main.lua` | Alternative entry point name (used by GUI-focused mods like GarageMenu, MenuCalculator) |
| `modIcon.dds` | In-game mod icon |

### `scripts/` — Lua Scripts

All Lua code lives under `scripts/`, organized into subdirectories by function.

```
scripts/
├── events/          # Network event classes
├── gui/             # GUI Lua code (paired with XML in gui/ or scripts/gui/)
├── utils/           # Shared utility modules
├── modLib/          # Shared library code (e.g., LogHelper, ModHelper)
└── <Feature>.lua    # Feature-specific modules
```

### `language/` — Translations

Translation files use the `l10n_<lang>.xml` naming convention:

```
language/
├── l10n_en.xml
├── l10n_de.xml
├── l10n_fr.xml
└── ...
```

Referenced in modDesc.xml via:
```xml
<l10n filenamePrefix="language/l10n" />
```

### `gui/` — GUI Code

GUI Lua and XML files live together:

```
gui/
├── MyMenu.lua
├── MyMenu.xml
└── guiProfiles.xml
```

### `events/` — Network Events

Network event classes can live at root (simple mods) or under `scripts/events/` (larger mods):

```
scripts/events/
├── MyEvent.lua
└── AnotherEvent.lua
```

### Other Asset Directories

| Directory | Purpose |
|---|---|
| `textures/` | DDS texture files |
| `sounds/` | OGG audio files |
| `drawing/` | I3D models, shaders, drawing assets |
| `triggers/` | Trigger I3D files |
| `xml/` | XML configuration data |
| `icons/` | Shop icons and other UI icons |
| `images/` | Menu icons and images |

---

## Registration Pattern

The entry point (`register.lua`) follows a consistent pattern:

### 1. Load All Scripts via `source()`

```lua
-- register.lua
source(Utils.getFilename("scripts/MyMod.lua", g_currentModDirectory))
source(Utils.getFilename("scripts/Utils/MyUtils.lua", g_currentModDirectory))
source(Utils.getFilename("scripts/Events/MyEvent.lua", g_currentModDirectory))
source(Utils.getFilename("scripts/Settings.lua", g_currentModDirectory))
```

Load in dependency order — dependencies first, dependents last.

### 2. Register Specializations

```lua
if g_specializationManager:getSpecializationByName("MyMod") == nil then
    g_specializationManager:addSpecialization(
        "myMod",
        "MyMod",
        Utils.getFilename("scripts/MyMod.lua", g_currentModDirectory),
        nil
    )
end
```

### 3. Inject into Vehicle Types via `TypeManager.validateTypes`

```lua
TypeManager.validateTypes = Utils.appendedFunction(TypeManager.validateTypes, function(self)
    if self.typeName == "vehicle" then
        MyModManager.injectSpecialisation()
    end
end)
```

### 4. Register Mod Event Listener

```lua
MyModManager = {}
addModEventListener(MyModManager)
```

### 5. Hook Game Managers

```lua
TypeManager.validateTypes = Utils.prependedFunction(TypeManager.validateTypes, MyModValidateVehicleTypes)
```

---

## modDesc.xml Conventions

### Standard Structure

```xml
<?xml version="1.0" encoding="utf-8" standalone="no" ?>
<modDesc descVersion="106">
    <author>YOUR NAME</author>
    <version>1.0.0.0</version>

    <title>
        <en>My Mod</en>
    </title>

    <description>
        <en><![CDATA[
        Description of the mod.
        ]]></en>
    </description>

    <multiplayer supported="true" />
    <iconFilename>modIcon.dds</iconFilename>

    <extraSourceFiles>
        <sourceFile filename="register.lua" />
    </extraSourceFiles>

    <l10n filenamePrefix="language/l10n" />

    <actions>
        <action name="MYMOD_TOGGLE" category="VEHICLE" axisType="HALF" ignoreComboMask="false" />
    </actions>

    <inputBinding>
        <actionBinding action="MYMOD_TOGGLE">
            <binding device="KB_MOUSE_DEFAULT" input="KEY_lshift KEY_r" />
        </actionBinding>
    </inputBinding>
</modDesc>
```

### Key Points

- **`<extraSourceFiles>` lists entry points only** — the entry point loads all other scripts via `source()`. This is the pattern used by AutoDrive, UniversalAutoload, TipAnywhere, and others.
- **`<l10n filenamePrefix>`** points to the translation directory prefix.
- **`<actions>` + `<inputBinding>`** define key bindings.
- **`descVersion`** varies by mod (99-110 in references) — use the latest supported by your target game version.

---

## Module Conventions

### Namespace Pattern

- Global namespace: `<ModName>` (e.g., `MyMod`)
- Sub-modules: `<ModName><ModuleName>` (e.g., `MyModSettings`, `MyModLogger`)
- Each module file creates its own table and does **not** pollute the global namespace beyond its own table

### Module Lifecycle

Every system module should implement a consistent lifecycle:

| Function | Purpose |
|---|---|
| `loadMap()` | Initialize on map load |
| `deleteMap()` | Cleanup on map delete |
| `update(dt)` | Per-frame logic |
| `draw()` | Render HUD elements |
| `keyEvent()` | Handle key input |

### Code Style

- **Indentation**: 4 spaces (tabs in some mods — be consistent)
- **Functions/methods**: `camelCase`
- **Classes**: `PascalCase`
- **Constants**: `UPPER_SNAKE_CASE`
- **Section headers**: `-- ============================================================= --` comment blocks
- **Semicolons**: optional but be consistent within a file

---

## Development Workflow

### Adding a New System

1. Create `scripts/<systemName>.lua` (or `scripts/<systemName>/<systemName>.lua` for complex systems)
2. Define the module table: `MyMod<SystemName> = {}`
3. Implement lifecycle functions (`loadMap`, `update`, `deleteMap`, etc.)
4. Add the file to `register.lua` via `source()` in dependency order
5. Register the module with `addModEventListener()` if it needs lifecycle events

### Adding a New Event

1. Create `scripts/events/<eventName>.lua`
2. Define the event class following the `Class(MyModEvent, Event)` pattern
3. Implement `emptyNew`, `new`, `readStream`, `writeStream`, `run`, and `sendEvent`
4. Add the file to `register.lua` via `source()`

### Adding a New Setting

1. Add the default to the settings module (e.g., `MyModSettings.DEFAULTS`)
2. Add a getter/setter if needed
3. If it needs a UI control, add it in the settings UI module
4. Add the translation string to `language/l10n_en.xml`

### Adding a New Translation

1. Add the text entry to `language/l10n_en.xml`:
   ```xml
   <text name="myModSettingName" text="My Setting Label" />
   ```
2. Add the same entry to other language files (e.g., `l10n_de.xml`)
3. Reference it in code via `g_i18n:getText("myModSettingName")`

### Adding a New GUI Element

1. Create the GUI module in `gui/` (Lua + XML)
2. Load it via `g_gui:loadGui()` in the `loadMap()` lifecycle
3. Add the file to `register.lua` via `source()`

---

## Quick Reference

| Question | Answer |
|---|---|
| Where does the entry point go? | `register.lua` (or `main.lua`) at mod root |
| Where do Lua scripts go? | `scripts/` |
| Where do network events go? | `scripts/events/` |
| Where does GUI code go? | `gui/` |
| Where do translations go? | `language/` (or `l10n/`, `translations/`) |
| Where do textures go? | `textures/` |
| Where do sounds go? | `sounds/` |
| Where do I3D models go? | `drawing/` or `triggers/` |
| How do I load scripts? | `source(Utils.getFilename("scripts/MyMod.lua", g_currentModDirectory))` in `register.lua` |
| How do I register a specialization? | `g_specializationManager:addSpecialization()` in `register.lua` |
| How do I inject into vehicle types? | Hook `TypeManager.validateTypes` with `Utils.appendedFunction()` |
| How do I get mod lifecycle events? | `addModEventListener(MyMod)` |
| What namespace should I use? | `<ModName>` global, `<ModName><Module>` for sub-modules |