# WebGPU Three.js TSL Skill

An Agent Skill for developing WebGPU-enabled Three.js applications using TSL (Three.js Shading Language).

## Overview

This skill provides Claude with comprehensive knowledge for:

- Setting up Three.js with WebGPU renderer
- Writing shaders using TSL (Three.js Shading Language)
- Creating node-based materials
- Building GPU compute shaders
- Implementing post-processing effects
- Integrating custom WGSL code

## Installation

### Claude Code

```bash
# Install from this repository
/skill install webgpu-threejs-tsl@<your-github-username>/webgpu-claude-skill
```

### Manual Installation

Copy the `skills/webgpu-threejs-tsl` folder to:
- **Global**: `~/.claude/skills/`
- **Project**: `<project>/.claude/skills/`

## Skill Contents

```
skills/webgpu-threejs-tsl/
└── SKILL.md          # Complete TSL and WebGPU documentation
```

## Topics Covered

- **WebGPU Setup**: Renderer initialization, scene setup
- **TSL Syntax**: Types, vectors, uniforms, operators
- **Node Materials**: All material types and properties
- **Custom Functions**: `Fn()` pattern for reusable shader logic
- **Control Flow**: Conditionals, loops, discard
- **Texturing**: Basic and advanced texture operations
- **Compute Shaders**: GPU compute with instanced arrays
- **Post-Processing**: Built-in effects and custom pipelines
- **WGSL Integration**: Custom WGSL function injection

## Resources

- [Three.js Shading Language Wiki](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language)
- [Three.js WebGPU Examples](https://github.com/mrdoob/three.js/tree/master/examples)
- [Agent Skills Specification](https://github.com/anthropics/skills)

## License

MIT License

Code examples derived from [Three.js](https://github.com/mrdoob/three.js) (MIT License).
