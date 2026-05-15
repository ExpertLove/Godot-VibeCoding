# 🎮 Godot VibeCoding — AI Agent Skills Collection

[![Godot 4.3+](https://img.shields.io/badge/Godot-4.3+-478CBF?logo=godot-engine&logoColor=white)](https://godotengine.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A comprehensive collection of **Godot 4.3+ AI agent skills** designed for vibe-coded game development. These skills provide structured, reusable knowledge for AI coding agents to build high-quality Godot games with best practices baked in.

---

## 📋 Skills Overview

### 🧩 Core & Fundamentals

| Skill | Description |
|-------|------------|
| [**godot-project-setup**](./skills/godot-project-setup/SKILL.md) | Scaffolds recommended directory structure, project settings, autoloads, and `.gitignore` for a new Godot project |
| [**scene-organization**](./skills/scene-organization/SKILL.md) | Best practices for scene tree design — composition vs inheritance, when to split scenes, node hierarchy patterns |
| [**component-system**](./skills/component-system/SKILL.md) | Reusable node components — composition patterns, component communication, and interface design |
| [**dependency-injection**](./skills/dependency-injection/SKILL.md) | Managing dependencies between systems — autoloads, service locators, `@export` injection, and scene injection patterns |
| [**event-bus**](./skills/event-bus/SKILL.md) | Decoupled communication between nodes — global EventBus autoload with typed signals |

### 🎨 2D & 3D Graphics

| Skill | Description |
|-------|------------|
| [**2d-essentials**](./skills/2d-essentials/SKILL.md) | TileMaps, parallax scrolling, 2D lights and shadows, canvas layers, particles 2D, custom drawing, and 2D meshes |
| [**3d-essentials**](./skills/3d-essentials/SKILL.md) | Materials, lighting, shadows, environment, global illumination, fog, LOD, occlusion culling, and decals |
| [**shader-basics**](./skills/shader-basics/SKILL.md) | Godot shader language, visual shaders, common visual recipes, and post-processing effects |
| [**particles-vfx**](./skills/particles-vfx/SKILL.md) | GPU particles, ParticleProcessMaterial, emission shapes, subemitters, trails, attractors, collision, and VFX recipes |

### 🎬 Animation

| Skill | Description |
|-------|------------|
| [**animation-system**](./skills/animation-system/SKILL.md) | AnimationPlayer, AnimationTree, blend trees, state machines, sprite animation, and code-driven animation |
| [**tween-animation**](./skills/tween-animation/SKILL.md) | Tweens — property animation, method tweening, chaining, parallel sequences, easing, and common motion recipes |
| [**godot-animator**](./skills/godot-animator.md) | AI agent skill for creating, managing, and optimizing animations in Godot 4 |

### 🧠 AI & Navigation

| Skill | Description |
|-------|------------|
| [**ai-navigation**](./skills/ai-navigation/SKILL.md) | NavigationAgent2D/3D, steering behaviors, behavior trees, and patrol patterns |
| [**state-machine**](./skills/state-machine/SKILL.md) | Enum-based, node-based, and resource-based FSM patterns with trade-offs |

### 🎮 Player & Input

| Skill | Description |
|-------|------------|
| [**player-controller**](./skills/player-controller/SKILL.md) | CharacterBody2D/3D patterns, input handling, physics, common movement recipes |
| [**input-handling**](./skills/input-handling/SKILL.md) | InputEvent system, Input Map actions, controllers/gamepads, mouse/touch, action rebinding |
| [**camera-system**](./skills/camera-system/SKILL.md) | Smooth follow, screen shake, camera zones, and transitions for 2D and 3D |

### 🔊 Audio

| Skill | Description |
|-------|------------|
| [**audio-system**](./skills/audio-system/SKILL.md) | Audio buses, AudioStreamPlayer, spatial audio, music management, SFX pooling, and dynamic mixing |

### 🖥️ UI & HUD

| Skill | Description |
|-------|------------|
| [**godot-ui**](./skills/godot-ui/SKILL.md) | Control nodes, themes, anchors, containers, and layout patterns |
| [**godot-ui-designer**](./skills/godot-ui-designer.md) | AI agent skill for building beautiful, responsive UIs in Godot 4 |
| [**responsive-ui**](./skills/responsive-ui/SKILL.md) | Multiple resolutions — stretch modes, aspect ratios, DPI scaling, and mobile/desktop adaptation |
| [**hud-system**](./skills/hud-system/SKILL.md) | Health bars, score displays, minimap, notifications, and damage numbers |

### 📦 Data & Systems

| Skill | Description |
|-------|------------|
| [**resource-pattern**](./skills/resource-pattern/SKILL.md) | Custom Resources for configuration, items, stats, and editor integration |
| [**save-load**](./skills/save-load/SKILL.md) | ConfigFile, JSON, Resource serialization, save game architecture |
| [**inventory-system**](./skills/inventory-system/SKILL.md) | Resource-based items, slot management, stacking, and UI binding |
| [**dialogue-system**](./skills/dialogue-system/SKILL.md) | Branching dialogue, conditions, variable interpolation, and UI presentation |

### 📐 Math & Physics

| Skill | Description |
|-------|------------|
| [**math-essentials**](./skills/math-essentials/SKILL.md) | Vectors, transforms, interpolation, curves, random number generation, and geometric recipes |
| [**physics-system**](./skills/physics-system/SKILL.md) | Physics bodies, collision shapes, raycasting, areas, rigid bodies, ragdolls, soft bodies, Jolt physics |

### 🌐 Multiplayer & Networking

| Skill | Description |
|-------|------------|
| [**multiplayer-basics**](./skills/multiplayer-basics/SKILL.md) | MultiplayerAPI, ENet/WebSocket peers, RPCs, and authority model |
| [**multiplayer-sync**](./skills/multiplayer-sync/SKILL.md) | MultiplayerSynchronizer, interpolation, prediction, and lag compensation |
| [**dedicated-server**](./skills/dedicated-server/SKILL.md) | Headless export, server architecture, lobby management, and deployment |

### 🛠️ Development Tools

| Skill | Description |
|-------|------------|
| [**addon-development**](./skills/addon-development/SKILL.md) | EditorPlugin, `@tool` scripts, custom inspectors, and dock panels |
| [**godot-tools-engineer**](./skills/godot-tools-engineer.md) | AI agent skill for building editor tools and plugins in Godot 4 |
| [**godot-code-review**](./skills/godot-code-review/SKILL.md) | Best practices checklist, common anti-patterns, and Godot-specific pitfalls |
| [**godot-code-reviewer**](./skills/godot-code-reviewer.md) | AI agent skill for reviewing GDScript code against best practices |
| [**godot-debugging**](./skills/godot-debugging/SKILL.md) | Remote debugger, print techniques, signal tracing, common error patterns |
| [**godot-testing**](./skills/godot-testing/SKILL.md) | TDD workflow with GUT and gdUnit4, covers both GDScript |
| [**godot-optimization**](./skills/godot-optimization/SKILL.md) | Profiler, draw calls, physics tuning, memory management, and common bottlenecks |
| [**godot-performance-profiler**](./skills/godot-performance-profiler.md) | AI agent skill for profiling and optimizing Godot 4 game performance |

### 🎨 Content Pipeline

| Skill | Description |
|-------|------------|
| [**assets-pipeline**](./skills/assets-pipeline/SKILL.md) | Image compression, 3D scene import, audio formats, resource formats, and import configuration |
| [**export-pipeline**](./skills/export-pipeline/SKILL.md) | Export presets, platform settings, CI/CD with GitHub Actions |
| [**localization**](./skills/localization/SKILL.md) | TranslationServer, CSV/PO translation files, locale switching, RTL support |

### 🌍 Advanced Systems

| Skill | Description |
|-------|------------|
| [**procedural-generation**](./skills/procedural-generation/SKILL.md) | Noise-based terrain, BSP dungeons, cellular automata caves, wave function collapse |
| [**godot-brainstorming**](./skills/godot-brainstorming/SKILL.md) | Guides scene tree planning, node type selection, and architectural decisions |
| [**godot-game-architect**](./skills/godot-game-architect.md) | AI agent skill for architecting complete Godot 4 game projects |
| [**godot-game-dev**](./skills/godot-game-dev.md) | AI agent skill for developing complete game features in Godot 4 |
| [**godot-shader-author**](./skills/godot-shader-author.md) | AI agent skill for writing shaders and visual effects in Godot 4 |
| [**using-godot-prompter**](./skills/using-godot-prompter/SKILL.md) | Bootstrap skill for finding and using GodotPrompter skills |

### 🥽 XR Development

| Skill | Description |
|-------|------------|
| [**xr-development**](./skills/xr-development/SKILL.md) | OpenXR setup, XROrigin3D, hand tracking, controllers, passthrough, and Meta Quest deployment |

---

## 🚀 Usage

### With KiloCode / Cline / Continue.dev

These skills are designed to be used with AI coding agents (KiloCode, Cline, Continue.dev, etc.). Each skill is a markdown file that an agent can load to gain specialized knowledge about a specific Godot 4 domain.

### Manual Installation

1. **Clone this repository** into your Godot project:
   ```bash
   git clone https://github.com/ExpertLove/godot_vibecoding.git .kilocode/skills/
   ```

2. **Or copy individual skills** into your AI agent's skills directory:
   ```
   your-project/
   └── .kilocode/
       └── skills/
           ├── 2d-essentials/
           ├── ai-navigation/
           ├── audio-system/
           └── ... (pick what you need)
   ```

3. **The agent will automatically** discover and use the relevant skill when you ask it to work on a specific area.

### As a Reference Library

Even without an AI agent, these files serve as an excellent **structured reference documentation** for Godot 4 development. Browse the `skills/` directory for any topic you need.

---

## 🏗️ Skill Structure

Each skill follows a consistent structure:

```
skill-name/
├── SKILL.md              # Main skill file — comprehensive guide
├── references/           # Advanced/deep-dive reference files
│   ├── advanced-topic.md
│   └── ...
```

The `SKILL.md` files contain:
- **Core concepts** and best practices
- **Code examples** with complete GDScript snippets
- **Anti-patterns** to avoid
- **Trade-offs** between different approaches
- **Cross-references** to related skills

---

## 📚 Reference Files

Many skills include **reference files** in the `references/` subdirectory for deeper dives:

| Skill | References |
|-------|-----------|
| 2d-essentials | TileMap, parallax, lights/shadows, custom drawing, 2D particles |
| 3d-essentials | Materials/lighting, decals, environment/post, fog, GI, LOD/culling |
| animation-system | IK recipes, bone constraints, retargeting, skeleton modifiers, sprite animation |
| ai-navigation | Behavior trees, steering behaviors, chase/attack, patrol patterns |
| audio-system | Audio settings, interactive music, music manager, SFX pooling |
| camera-system | Camera3D patterns, camera zones, split-screen, transitions |
| godot-debugging | Performance debugging, scene tree debugging, signal tracing |
| godot-optimization | Draw calls, CPU bottlenecks, memory management, physics tuning |
| godot-testing | GUT reference, gdUnit4 reference, TDD workflow, testing patterns |
| input-handling | Action rebinding, gamepad, mouse, touch |
| inventory-system | Equipment, serialization, UI binding |
| multiplayer-basics | Disconnect handling, player join flow, spawning networked objects |
| multiplayer-sync | Bandwidth optimization, client prediction, interpolation, lag compensation |
| physics-system | Raycasting, rigidbody, staticbody, area, ragdoll, softbody recipes |
| procedural-generation | BSP dungeons, cellular automata, noise generation, WFC |

---

## 🎯 Why VibeCoding?

These skills are specifically designed for **vibe-coded** game development — where you describe features in natural language and your AI agent implements them. The skills provide:

- ✅ **Structured knowledge** — AI agents don't need to guess Godot APIs
- ✅ **Best practices** — Production-ready patterns, not toy examples
- ✅ **Godot 4.3+ optimized** — Latest APIs and techniques
- ✅ **Comprehensive coverage** — From project setup to shipping

---

## 🤝 Contributing

Contributions are welcome! If you have improvements, bug fixes, or new skills to add:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-skill`)
3. Commit your changes (`git commit -m 'Add amazing skill'`)
4. Push to the branch (`git push origin feature/amazing-skill`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Built for the **Godot 4.3+** engine
- Designed for AI-assisted game development workflows
- Inspired by the Godot community's collective knowledge

---

<p align="center">
  Made with ❤️ for the Godot VibeCoding community
</p>
