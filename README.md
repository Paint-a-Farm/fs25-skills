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

- **fs25-mod-deploy** — Mod packaging, symlinks, modDesc.xml standards, platform-specific paths (Windows/macOS)
- **fs25-scripting-mod** — Lua scripting patterns, specializations, events, base game script reference
- **fs25-fs-utils** — CLI tools for decompiling scripts, extracting GAR/DLC archives, unlocking shapes, formatting XML

## License

MIT
