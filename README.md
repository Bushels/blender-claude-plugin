# Blender Skills for AI Coding Agents

Expert skills for Blender 5.x Python scripting (targeting 5.0 and 5.1). Provides complete API references, node/modifier/constraint catalogs, recipes, and debugging guides across 8 domains.

## Skills

| Skill                          | Description                                                                                                  |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| **blender-geometry-nodes**     | ~373 geometry nodes, procedural modeling, Bone Info node, String to Curves fields, volume grid nodes, SVD       |
| **blender-shader-nodes**       | ~95 shader nodes, PBR/procedural/glass/metal recipes, Raycast node, Normal Map OpenGL/DirectX (5.1)            |
| **blender-compositing-nodes**  | ~80 compositor nodes, denoise/color grading/keying, Sequencer Strip Info, Mask to SDF (5.1)                    |
| **blender-python-scripting**   | Operators, panels, add-ons, handlers, timers, property system, Python 3.13 (5.1)                               |
| **blender-animation-rigging**  | Keyframes, FCurves, Smooth Gaussian modifier (5.1), Apply to Basis, drivers, ~45 constraints, IK/FK, NLA        |
| **blender-modeling-modifiers** | ~50 modifiers, bmesh API, boolean speedups (5.1), Corrective Flip Normals, hard surface/retopo workflows       |
| **blender-physics-simulation** | Rigid body, cloth, fluid (Mantaflow), soft body, particles, force fields, baking                                |
| **blender-scene-rendering**   | Cycles/EEVEE, AVIF output (5.1), HTJ2K EXR, OpenColorIO 2.5, color management, import/export, linking          |

## Install

### Claude Code

Add the marketplace and install the plugin:

```bash
claude plugin marketplace add ra100/blender-claude-plugin
claude plugin install blender-skills@blender-claude-marketplace
```

Or from a local checkout:

```bash
claude plugin marketplace add /path/to/blender-claude-plugin
claude plugin install blender-skills@blender-claude-marketplace
```

### Manual

Copy the plugin directory into the agent's plugin search path. The agent needs to read `.claude-plugin/plugin.json` to discover the plugin and `skills/*/SKILL.md` to activate individual skills.

### Blender MCP Server (recommended)

The skills are designed to work alongside the **official Blender MCP Server** (Blender 5.1+). Once configured, Claude can drive Blender directly instead of emitting scripts to copy-paste. See [docs/blender-mcp-setup.md](docs/blender-mcp-setup.md) for the full setup. Skills work without it — they fall back to generating runnable `bpy` scripts.

## Usage

The skills activate automatically based on your prompt. Ask Claude to:

- Create a geometry node scatter system
- Write a PBR material with Python
- Set up a compositor denoise + color grading pipeline
- Build a custom Blender operator with undo support
- Rig a character with IK/FK switching
- Add a subdivision + bevel + mirror modifier stack
- Simulate cloth falling onto rigid bodies
- Configure Cycles GPU rendering with denoising
- Export a character as glTF with animations

Each skill includes a task decision tree, common recipes, debugging guides, and full Python API references with correct type strings.

## Structure

```
skills/
├── blender-geometry-nodes/
│   ├── SKILL.md
│   └── references/
│       ├── node_reference.md
│       └── python_api.md
├── blender-shader-nodes/
│   ├── SKILL.md
│   └── references/
│       ├── node_reference.md
│       └── python_api.md
├── blender-compositing-nodes/
│   ├── SKILL.md
│   └── references/
│       ├── node_reference.md
│       └── python_api.md
├── blender-python-scripting/
│   ├── SKILL.md
│   └── references/
│       ├── python_api.md
│       └── data_model_reference.md
├── blender-animation-rigging/
│   ├── SKILL.md
│   └── references/
│       ├── constraint_reference.md
│       └── python_api.md
├── blender-modeling-modifiers/
│   ├── SKILL.md
│   └── references/
│       ├── modifier_reference.md
│       └── python_api.md
├── blender-physics-simulation/
│   ├── SKILL.md
│   └── references/
│       ├── physics_reference.md
│       └── python_api.md
└── blender-scene-rendering/
    ├── SKILL.md
    └── references/
        ├── settings_reference.md
        └── python_api.md
```

## Changelog

### v1.3.0 — Official Blender MCP Server

- Re-targeted MCP integration from third-party `ahujasid/blender-mcp` to the **official Blender Lab MCP Server** (Blender 5.1+, <https://www.blender.org/lab/mcp-server/>).
- Added `docs/blender-mcp-setup.md` with end-to-end setup (add-on install, MCP server install, Claude Code wiring, headless mode, troubleshooting).
- Each `SKILL.md` MCP-First section now lists concrete official tool names (`execute_blender_code`, `get_objects_summary`, `search_api_docs`, screenshots, jump-to, render-to-path, …) and a recommended inspect → mutate → verify workflow.
- README and `AGENTS.md` updated with the new architecture and tool catalog.

### v1.2.0 — Blender 5.1 Updates

- **Geometry Nodes**: Bone Info node, String to Curves field inputs + Word output, volume grid nodes (Cube Grid, Clip Grid, Grid Mean/Median, Grid to Points, Grid Dilate/Erode), UV Unwrap Minimum Stretch (SLIM), Pack UV Islands custom region, Matrix SVD node, Get/Store Bundle Item nodes, Font socket type, node warning search (Ctrl+F), node tools as operators
- **Shader Nodes**: Raycast shader node, Normal Map OpenGL/DirectX toggle, Normal Map on displaced mesh (Cycles)
- **Compositing Nodes**: Sequencer Strip Info node, Mask to SDF node, Mix node alpha performance (up to 2x)
- **Animation & Rigging**: Smooth (Gaussian) FCurve modifier, Apply to Basis operator, layered actions performance improvements, removed `ANIM_OT_convert_legacy_action`
- **Python Scripting**: Python 3.13 upgrade, type hint syntax fully supported, removed operators documented
- **Modeling & Modifiers**: Boolean modifier speed improvements, Corrective Flip Normals operator, snap to face center, loop selection delimiter improvements
- **Scene & Rendering**: AVIF output format, OpenEXR HTJ2K codec, video custom CRF, OpenColorIO 2.5, Light Path in World node trees, Cycles/EEVEE performance improvements
- **Physics**: No major API changes in 5.1

### v1.1.0 — Initial Release

- 8 expert skills covering all major Blender domains
- Targets Blender 5.0
- Complete node/modifier/constraint catalogs with Python type strings
- MCP-first approach with Python script fallback

## MCP Integration (Official Blender MCP Server)

All skills prefer the **official [Blender MCP Server](https://www.blender.org/lab/mcp-server/)** by Blender Lab when available for direct Blender interaction. They fall back to generating Python scripts when MCP is unavailable.

Setup at a glance:

1. **Install Blender 5.1+ add-on** — drag-drop or zip install from <https://projects.blender.org/lab/blender_mcp/releases>. Add-on runs a TCP bridge on `localhost:9876`.
2. **Install MCP server** — `pip install "git+https://projects.blender.org/lab/blender_mcp"` or use the `.mcpb` bundle.
3. **Wire into Claude Code** — add `blender-mcp` to `mcpServers` in `~/.claude/settings.json`.

Full guide: [docs/blender-mcp-setup.md](docs/blender-mcp-setup.md).

> ⚠️ The MCP server executes LLM-generated Python in your Blender session with no sandbox. Use a VM or a workstation without sensitive data.

Tools the skills use:

- `execute_blender_code` — run `bpy` Python in Blender
- `get_objects_summary`, `get_object_detail_summary` — scene inspection
- `get_blendfile_summary_*` — datablocks, missing files, linked libraries, paths
- `search_api_docs`, `get_python_api_docs`, `search_manual_docs` — bundled docs
- `get_screenshot_of_*`, `jump_to_*` — UI inspection / navigation
- `render_thumbnail_to_path`, `render_viewport_to_path` — quick renders

## License

[MIT](LICENSE)
