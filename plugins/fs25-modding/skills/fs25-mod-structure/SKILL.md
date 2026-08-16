---
name: fs25-mod-structure
description: Use when structuring FS25 mod projects, organizing Lua script files, setting up modular architecture, or implementing standardized logging.
---

# FS25 Mod Structure & Organization

## Overview

A well-organized FS25 mod is modular, maintainable, and future-proof. This skill defines a standardized project hierarchy, file naming conventions, module architecture, and logging standards that apply to **any** FS25 mod — vehicle mods, map mods, placeables, or pure script mods.

The goal is complete modularity: each Lua file has a single, well-defined responsibility, and the project structure makes it obvious where new code belongs.

## When to Use

- Setting up a new FS25 mod project structure
- Refactoring an existing mod for better organization
- Adding new systems, events, settings, or GUI elements to a mod
- Implementing standardized logging
- Deciding where a new Lua file should live

## Standardized Project Hierarchy

```
<ModName>/
├── .gitattributes              # Git line-ending normalization
├── artwork/                    # Mod icons, promotional images, banners
├── src/
│   ├── modDesc.xml             # Mod descriptor (metadata, script entry points)
│   ├── modIcon.dds             # In-game mod icon
│   ├── l10n/                   # Language / translation files
│   │   └── l10n_en.xml
│   ├── i3d/                    # Models and shapes
│   │   ├── myModel.i3d
│   │   └── myModel.i3d.shapes
│   └── data/                   # All Lua script files
│       ├── core/               # Core framework: namespace, main spec, registration
│       ├── systems/            # Feature systems (one subdirectory per system)
│       ├── events/             # Network event classes
│       ├── settings/           # Settings management & in-game settings UI
│       ├── gui/                # GUI / HUD / menu code
│       ├── logmanager/         # Centralized logging
│       ├── utils/              # Shared utility modules
│       └── config/             # Configuration data (defaults, constants)
```

### Top-Level Directories

| Directory | Purpose |
|---|---|
| `artwork/` | Mod icons, promotional images, banners used in modDesc or documentation |
| `src/` | All mod content: descriptor, models, translations, and scripts |
| `src/l10n/` | Language/translation XML files (`l10n_en.xml`, `l10n_es.xml`, etc.) |
| `src/i3d/` | I3D model files and their `.shapes` companions |
| `src/data/` | All Lua script files, organized into functional subdirectories |

---

## `src/data/` Subdirectory Reference

Each subdirectory under `src/data/` has a specific purpose. Use the table below to determine where a new file belongs.

### `core/` — Core Framework

**Purpose**: The mod's entry point, namespace definition, main specialization, and registration logic.

**Use Case**: Files that bootstrap the mod — the main specialization table, the registration file that hooks into the game's type/specialization managers, and the namespace definition.

**Setup Example**:

```
src/data/core/
├── MyMod.lua          # Main namespace + specialization
└── register.lua       # Registration: adds specialization, hooks managers
```

`MyMod.lua`:
```lua
--[[
    MY MOD - MAIN NAMESPACE
    =============================================================
    Defines the global namespace and main specialization.
]]

MyMod = MyMod or {};

-- Configuration defaults
MyMod.DEBUG_MODE = false;
MyMod.ENABLE_MY_SYSTEM = true;

-- Debug print helper (only prints when DEBUG_MODE is enabled)
function MyMod.debugPrint(...)
    if not MyMod.DEBUG_MODE then
        return;
    end
    local args = {...};
    for i = 1, #args do
        args[i] = tostring(args[i]);
    end
    print("[MyMod]: " .. table.concat(args, " "));
end
```

`register.lua`:
```lua
--[[
    MY MOD - REGISTRATION
    =============================================================
    Registers the specialization and hooks into game managers.
]]

local modDirectory = g_currentModDirectory;

if g_specializationManager ~= nil then
    g_specializationManager:addSpecialization(
        'myMod',
        'MyMod',
        Utils.getFilename('src/data/core/MyMod.lua', modDirectory),
        ""
    );
end
```

---

### `systems/` — Feature Systems

**Purpose**: Self-contained feature systems. Each system gets its **own subdirectory** with all files related to that feature.

**Use Case**: Any distinct gameplay feature — weather effects, crop growth, vehicle mechanics, production systems, etc. Each system should be independently loadable and have a clear lifecycle (`init`, `update`, `cleanup`).

**Setup Example**:

```
src/data/systems/
├── mySystem/
│   ├── mySystem.lua        # System logic
│   └── mySystemSpec.lua    # Spec field initialization (if vehicle-based)
└── anotherSystem/
    └── anotherSystem.lua
```

`mySystem.lua`:
```lua
--[[
    MY MOD - MY SYSTEM
    =============================================================
    Handles all logic for the "My System" feature.
]]

MyModMySystem = {};

-- Initialize system spec fields on a vehicle
function MyModMySystem.initSpecFields(vehicle, spec)
    if vehicle == nil or spec == nil then
        return;
    end
    spec.mySystemState = {};
    spec.mySystemLastUpdate = 0;
end

-- Per-frame update
function MyModMySystem.update(vehicle, spec, dt)
    if vehicle == nil or spec == nil then
        return;
    end
    -- System logic here
end

-- Cleanup on vehicle delete
function MyModMySystem.onDelete(vehicle, spec)
    if vehicle == nil or spec == nil then
        return;
    end
    -- Release resources
end
```

---

### `events/` — Network Events

**Purpose**: Network event classes for multiplayer synchronization.

**Use Case**: Any data that must be synced between server and clients — state changes, actions triggered by the mod, gameplay events.

**Setup Example**:

```
src/data/events/
├── myEvent.lua
└── anotherEvent.lua
```

`myEvent.lua`:
```lua
--[[
    MY MOD - MY EVENT
    =============================================================
    Network event for synchronizing a state change.
]]

MyModMyEvent = {};
local MyModMyEvent_mt = Class(MyModMyEvent, Event);
InitEventClass(MyModMyEvent, "MyModMyEvent");

function MyModMyEvent.emptyNew()
    local self = Event.new(MyModMyEvent_mt);
    return self;
end

function MyModMyEvent.new(vehicle, value)
    local self = MyModMyEvent.emptyNew();
    self.vehicle = vehicle;
    self.value = value;
    return self;
end

function MyModMyEvent:readStream(streamId, connection)
    self.vehicle = NetworkUtil.readNodeObject(streamId);
    self.value = streamReadFloat32(streamId);
    self:run(connection);
end

function MyModMyEvent:writeStream(streamId, connection)
    NetworkUtil.writeNodeObject(streamId, self.vehicle);
    streamWriteFloat32(streamId, self.value);
end

function MyModMyEvent:run(connection)
    if not connection:getIsServer() then
        -- Apply on client, then broadcast
        g_server:broadcastEvent(self, false, connection, self.vehicle);
    else
        -- Apply on server
        if self.vehicle ~= nil then
            self.vehicle:myFunction(self.value, true);
        end
    end
end

function MyModMyEvent.sendEvent(vehicle, value, noEventSend)
    if noEventSend == nil or not noEventSend then
        if g_server ~= nil then
            g_server:broadcastEvent(MyModMyEvent.new(vehicle, value), nil, nil, vehicle);
        else
            g_client:getServerConnection():sendEvent(MyModMyEvent.new(vehicle, value));
        end
    end
end
```

---

### `settings/` — Settings Management

**Purpose**: Settings storage, defaults, and in-game settings UI integration.

**Use Case**: Any mod with user-configurable options — toggles, sliders, dropdowns. Centralizes all settings so other systems read from one source of truth.

**Setup Example**:

```
src/data/settings/
├── settingsManager.lua    # Settings storage & access
└── settingsUI.lua         # In-game settings page integration
```

`settingsManager.lua`:
```lua
--[[
    MY MOD - SETTINGS MANAGER
    =============================================================
    Centralized settings storage and access.
]]

MyModSettings = {
    DEFAULTS = {
        enableMyFeature = true,
        myFactor = 1.0,
        mySpeed = 30
    },
    values = {}
};

-- Initialize settings from defaults
function MyModSettings:init()
    for key, default in pairs(MyModSettings.DEFAULTS) do
        self.values[key] = default;
    end
end

-- Get a boolean setting
function MyModSettings:getBool(name, fallback)
    if self.values[name] == nil then return fallback end
    return self.values[name] == true;
end

-- Get a numeric setting
function MyModSettings:getNumber(name, fallback)
    if self.values[name] == nil then return fallback end
    return self.values[name];
end

-- Set a setting value
function MyModSettings:set(name, value)
    self.values[name] = value;
end
```

---

### `gui/` — GUI / HUD / Menu Code

**Purpose**: All user interface code — HUD overlays, in-game menus, dialogs, and screen elements.

**Use Case**: Any mod that renders text, draws HUD elements, or injects into in-game menus (settings pages, shop screens, etc.).

**Setup Example**:

```
src/data/gui/
├── hud.lua            # HUD overlay rendering
└── myMenu.lua         # Custom menu / dialog
```

`hud.lua`:
```lua
--[[
    MY MOD - HUD
    =============================================================
    Renders HUD overlay elements.
]]

MyModHud = {
    textPosX = 0.99,
    textPosY = 0.85,
    textSize = 0.015 * g_gameSettings.uiScale
};

function MyModHud:draw(vehicle, spec)
    if g_currentMission.controlledVehicle == nil then
        return;
    end

    setTextColor(0, 1, 0, 1);
    setTextAlignment(RenderText.ALIGN_RIGHT);
    setTextVerticalAlignment(RenderText.VERTICAL_ALIGN_TOP);
    setTextBold(true);

    local hudLineLevel = self.textPosY;
    local hudText = string.format("My Mod: %.2f", spec.myValue or 0);
    renderText(self.textPosX, hudLineLevel, self.textSize, hudText);

    setTextColor(1, 1, 1, 1);
    setTextAlignment(RenderText.ALIGN_LEFT);
    setTextVerticalAlignment(RenderText.VERTICAL_ALIGN_BASELINE);
    setTextBold(false);
end
```

---

### `logmanager/` — Centralized Logging

**Purpose**: A single, standardized logging module used by all other systems.

**Use Case**: Every mod should have one logger. All systems call the logger instead of raw `print()`, giving consistent formatting, log levels, and easy debug toggling.

**Setup Example**:

```
src/data/logmanager/
└── logger.lua
```

`logger.lua`:
```lua
--[[
    MY MOD - LOGGER
    =============================================================
    Centralized logging with standardized levels and format.
]]

MyModLogger = {
    DEBUG_ENABLED = false,
    PREFIX = "[MyMod]"
};

-- Log levels
MyModLogger.LEVELS = {
    INFO = "INFO",
    WARN = "WARN",
    ERROR = "ERROR",
    DEBUG = "DEBUG"
};

-- Core log function
function MyModLogger:log(level, moduleName, message, ...)
    local args = {...};
    for i = 1, #args do
        args[i] = tostring(args[i]);
    end
    local msg = message;
    if #args > 0 then
        msg = string.format(message, table.unpack(args));
    end
    print(string.format("%s[%s] %s: %s", self.PREFIX, level, moduleName, msg));
end

-- Info level
function MyModLogger:info(moduleName, message, ...)
    self:log(self.LEVELS.INFO, moduleName, message, ...);
end

-- Warning level
function MyModLogger:warn(moduleName, message, ...)
    self:log(self.LEVELS.WARN, moduleName, message, ...);
end

-- Error level
function MyModLogger:error(moduleName, message, ...)
    self:log(self.LEVELS.ERROR, moduleName, message, ...);
end

-- Debug level (only when DEBUG_ENABLED)
function MyModLogger:debug(moduleName, message, ...)
    if not self.DEBUG_ENABLED then
        return;
    end
    self:log(self.LEVELS.DEBUG, moduleName, message, ...);
end
```

**Usage in other modules**:
```lua
MyModLogger:info("MySystem", "System initialized for vehicle %s", vehicleName);
MyModLogger:warn("MySystem", "Missing dependency: %s", dependencyName);
MyModLogger:error("MySystem", "Failed to load: %s", errorMessage);
MyModLogger:debug("MySystem", "State: %s", tostring(state));
```

---

### `utils/` — Utility Modules

**Purpose**: Shared, stateless helper functions used across multiple systems.

**Use Case**: Math helpers, string utilities, table utilities, vehicle helpers — anything that doesn't belong to a single feature system.

**Setup Example**:

```
src/data/utils/
├── mathUtils.lua
├── tableUtils.lua
└── vehicleUtils.lua
```

`mathUtils.lua`:
```lua
--[[
    MY MOD - MATH UTILITIES
    =============================================================
    Shared math helper functions.
]]

MyModMathUtils = {};

-- Clamp a value between min and max
function MyModMathUtils.clamp(value, min, max)
    return math.max(min, math.min(max, value));
end

-- Linear interpolation
function MyModMathUtils.lerp(a, b, t)
    return a + (b - a) * t;
end
```

`vehicleUtils.lua`:
```lua
--[[
    MY MOD - VEHICLE UTILITIES
    =============================================================
    Shared vehicle helper functions.
]]

MyModVehicleUtils = {};

-- Get all members of a vehicle train (root + attached implements/trailers)
function MyModVehicleUtils.getVehicleTrainMembers(rootVehicle)
    local members = {};
    if rootVehicle == nil then
        return members;
    end

    if rootVehicle.getChildVehicles ~= nil then
        local childVehicles = rootVehicle:getChildVehicles();
        if childVehicles ~= nil then
            for _, child in ipairs(childVehicles) do
                if child ~= nil then
                    members[child] = true;
                end
            end
            members[rootVehicle] = true;
            return members;
        end
    end

    -- Fallback: recursive walk
    local function addVehicle(vehicle)
        if vehicle == nil or members[vehicle] ~= nil then
            return;
        end
        members[vehicle] = true;

        if vehicle.getAttachedImplements ~= nil then
            local implements = vehicle:getAttachedImplements();
            if implements ~= nil then
                for _, impl in ipairs(implements) do
                    if impl ~= nil then
                        addVehicle(impl.object or impl);
                    end
                end
            end
        end

        if vehicle.getAttachedTrailers ~= nil then
            local trailers = vehicle:getAttachedTrailers();
            if trailers ~= nil then
                for _, trailer in ipairs(trailers) do
                    addVehicle(trailer);
                end
            end
        end
    end

    addVehicle(rootVehicle);
    return members;
end
```

---

### `config/` — Configuration Data

**Purpose**: Static configuration data — defaults, constants, lookup tables.

**Use Case**: Any mod with configurable data that users or modders may want to edit without touching system logic. Keeps data separate from code.

**Setup Example**:

```
src/data/config/
├── defaults.lua        # Default configuration values
└── constants.lua       # Mod-wide constants
```

`defaults.lua`:
```lua
--[[
    MY MOD - DEFAULTS
    =============================================================
    Default configuration values.
]]

MyModDefaults = {
    ENABLE_FEATURE = true,
    FACTOR = 1.0,
    THRESHOLD = 30
};
```

`constants.lua`:
```lua
--[[
    MY MOD - CONSTANTS
    =============================================================
    Mod-wide constants.
]]

MyModConstants = {
    CHECK_CYCLE_MS = 200,
    MIN_VALUE = 50,
    MAX_RATE = 0.5,
    STANDARD_RATE = 0.006
};
```

---

## File Naming & Module Conventions

### File Naming

- **CamelCase** for file names: `effectManager.lua`, `settingsManager.lua`, `weatherSystem.lua`
- **Single responsibility**: each file handles exactly one function/stream
- **No monolithic files**: if a file exceeds ~300 lines, consider splitting it into focused modules

### Namespace Pattern

- Global namespace: `<ModName>` (e.g., `MyMod`)
- Sub-modules: `<ModName><ModuleName>` (e.g., `MyModSettings`, `MyModLogger`, `MyModMySystem`)
- Each module file creates its own table and does **not** pollute the global namespace beyond its own table

### Module Lifecycle

Every system module should implement a consistent lifecycle:

| Function | Purpose |
|---|---|
| `initSpecFields(vehicle, spec)` | Initialize per-vehicle state |
| `update(vehicle, spec, dt)` | Per-frame logic |
| `onDelete(vehicle, spec)` | Cleanup resources |

### Code Style

- **Indentation**: 4 spaces
- **Functions/methods**: `camelCase`
- **Classes**: `PascalCase`
- **Constants**: `UPPER_SNAKE_CASE`
- **Section headers**: `--[[ ... ]]` comment blocks
- **Semicolons**: optional but be consistent within a file

---

## Standardized Logging

### Format

```
[<ModName>][LEVEL] ModuleName: message
```

### Levels

| Level | When to Use |
|---|---|
| `INFO` | Normal lifecycle events (loaded, initialized, registered) |
| `WARN` | Non-fatal issues (missing optional dependency, fallback used) |
| `ERROR` | Fatal issues (failed to load, nil critical value) |
| `DEBUG` | Detailed state dumps (only when debug mode enabled) |

### Rules

1. **Always use the logger** — never raw `print()` in system modules
2. **Include the module name** — makes log filtering easy
3. **Use `string.format`** for dynamic messages
4. **Debug messages are gated** behind `DEBUG_ENABLED` to avoid log spam
5. **One logger per mod** — defined in `src/data/logmanager/logger.lua`

---

## modDesc.xml Conventions

### Load Order

Load files in dependency order — dependencies first, dependents last:

```xml
<extraSourceFiles>
    <!-- 1. Core framework (namespace, logger, config) -->
    <sourceFile filename="src/data/logmanager/logger.lua"/>
    <sourceFile filename="src/data/config/constants.lua"/>
    <sourceFile filename="src/data/config/defaults.lua"/>

    <!-- 2. Utilities -->
    <sourceFile filename="src/data/utils/mathUtils.lua"/>
    <sourceFile filename="src/data/utils/vehicleUtils.lua"/>

    <!-- 3. Settings -->
    <sourceFile filename="src/data/settings/settingsManager.lua"/>

    <!-- 4. Systems (in dependency order) -->
    <sourceFile filename="src/data/systems/mySystem/mySystem.lua"/>

    <!-- 5. Events -->
    <sourceFile filename="src/data/events/myEvent.lua"/>

    <!-- 6. GUI -->
    <sourceFile filename="src/data/gui/hud.lua"/>

    <!-- 7. Registration + main specialization LAST -->
    <sourceFile filename="src/data/core/register.lua"/>
    <sourceFile filename="src/data/core/MyMod.lua"/>
</extraSourceFiles>
```

### Configuration Section

```xml
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

    <multiplayer supported="true"/>
    <iconFilename>modIcon.dds</iconFilename>
    <l10n filenamePrefix="l10n"/>

    <extraSourceFiles>
        <!-- ... -->
    </extraSourceFiles>

    <myModConfigurations
        debugMode="false"
        enableMyFeature="true">
    </myModConfigurations>
</modDesc>
```

---

## Development Workflow

### Adding a New System

1. Create `src/data/systems/<systemName>/<systemName>.lua`
2. Define the module table: `MyMod<SystemName> = {}`
3. Implement `initSpecFields`, `update`, and `onDelete` lifecycle functions
4. Add the file to `modDesc.xml` in the systems section
5. Call `initSpecFields` from the main specialization's init
6. Call `update` from the global update loop or `onUpdate`
7. Call `onDelete` from the main specialization's cleanup

### Adding a New Event

1. Create `src/data/events/<eventName>.lua`
2. Define the event class following the `Class(MyModEvent, Event)` pattern
3. Implement `emptyNew`, `new`, `readStream`, `writeStream`, `run`, and `sendEvent`
4. Add the file to `modDesc.xml` in the events section

### Adding a New Setting

1. Add the default to `MyModSettings.DEFAULTS` in `settingsManager.lua`
2. Add a getter/setter if needed
3. If it needs a UI control, add it in `settingsUI.lua`
4. Add the translation string to `src/l10n/l10n_en.xml`

### Adding a New Translation

1. Add the text entry to `src/l10n/l10n_en.xml`:
   ```xml
   <text name="myModSettingName" text="My Setting Label" />
   ```
2. Add the same entry to other language files (e.g., `l10n_es.xml`)
3. Reference it in code via `g_i18n:getText("myModSettingName")`

### Adding a New GUI Element

1. Create the GUI module in `src/data/gui/` (or add to an existing one)
2. Implement the draw/render function
3. Call it from the main specialization's `onDraw` event
4. Add the file to `modDesc.xml` in the GUI section

---

## Quick Reference

| Question | Answer |
|---|---|
| Where does the main specialization go? | `src/data/core/` |
| Where does registration go? | `src/data/core/register.lua` |
| Where do feature systems go? | `src/data/systems/<systemName>/` |
| Where do network events go? | `src/data/events/` |
| Where does settings management go? | `src/data/settings/` |
| Where does GUI/HUD code go? | `src/data/gui/` |
| Where does the logger go? | `src/data/logmanager/logger.lua` |
| Where do shared helpers go? | `src/data/utils/` |
| Where does config data go? | `src/data/config/` |
| Where do translations go? | `src/l10n/` |
| Where do models go? | `src/i3d/` |
| How should I log? | `MyModLogger:info("Module", "message")` |
| What namespace should I use? | `<ModName>` global, `<ModName><Module>` for sub-modules |