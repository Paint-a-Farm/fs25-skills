# FS25 Skills

AI agent skills for Farming Simulator 25 mod development — packaging, deployment, Lua scripting, and game file tools.

## Installation

### Claude Code

Add the marketplace:

```
/plugin marketplace add paint-a-farm/fs25-skills
```

Install the plugin:

```
/plugin install fs25-modding
```

### Cursor

1. Open Cursor Settings (Cmd+Shift+J / Ctrl+Shift+J)
2. Navigate to `Rules & Command` → `Project Rules` → Add Rule → Remote Rule (GitHub)
3. Enter: `https://github.com/paint-a-farm/fs25-skills.git`

Skills are automatically discovered and used by the agent based on context.

### Any agent

```
bunx skills add paint-a-farm/fs25-skills
```

## Skills Included

### fs25-modding

#### Project & Deployment

- **fs25-mod-deploy** — Mod packaging, symlinks, modDesc.xml standards, platform-specific paths
- **fs25-fs-utils** — CLI tools for decompiling scripts, extracting archives, unlocking shapes, formatting XML

#### Scripting & Configuration

- **fs25-scripting-mod** — Lua scripting patterns, specializations, events, base game script reference
- **fs25-vehicle-mod** — Vehicle XML structure, specializations, components/joints, movingTools/Parts
- **fs25-map-mod** — Map structure, terrain, farmlands, placeables, ground types

#### Assets & Formats

- **fs25-i3d-structure** — I3D file format, node types, materials, shapes, collision flags, XSD schema
- **fs25-texture-guide** — DDS formats, naming conventions, resolution guidelines, shader variations

#### Debugging

- **fs25-modding-debug** — Log reading, common errors, console commands, debugging checklist

## License

MIT
