# Compositor Effects (Godot 4.3+)

Reference for `skills/shader-basics/SKILL.md` — CompositorEffect setup, custom CompositorEffect (GDScript), built-in Compositor uses.

> ← Back to [SKILL.md](../SKILL.md)

---
## 7. Compositor Effects (Godot 4.3+)

The `Compositor` resource and `CompositorEffect` class let you insert custom render passes into the rendering pipeline. This is the official way to add screen-space effects, custom SSAO, outline passes, or any GPU compute work.

### When to Use Compositor vs Other Approaches

| Approach | Use For |
|----------|---------|
| ShaderMaterial on node | Per-object effects (dissolve, outline on one sprite) |
| CanvasLayer + ColorRect | Simple full-screen 2D post-processing |
| WorldEnvironment | Built-in 3D post-processing (glow, SSAO, DOF) |
| **CompositorEffect** | Custom render passes, compute shaders, effects not available in WorldEnvironment |

### Setup

1. Select your `WorldEnvironment` node (or `Camera3D`)
2. In Inspector → **Compositor** → New `Compositor`
3. In the Compositor → **Effects** → Add Element → New `CompositorEffect`
4. Set `Effect Callback Type` to when the effect runs in the pipeline:
   - `PRE_OPAQUE` — before opaque geometry
   - `POST_OPAQUE` — after opaque, before transparent
   - `POST_SKY` — after sky rendering
   - `PRE_TRANSPARENT` — before transparent geometry
   - `POST_TRANSPARENT` — after all geometry (most common for post-processing)

### Custom CompositorEffect (GDScript)

```gdscript
# custom_outline_effect.gd
@tool
class_name CustomOutlineEffect
extends CompositorEffect

func _init() -> void:
    effect_callback_type = EFFECT_CALLBACK_TYPE_POST_TRANSPARENT
    needs_normal_roughness = true  # request normal buffer access

func _render_callback(effect_callback_type: int, render_data: RenderData) -> void:
    var render_scene_data: RenderSceneData = render_data.get_render_scene_data()
    var render_scene_buffers: RenderSceneBuffers = render_data.get_render_scene_buffers()

    if not render_scene_buffers:
        return

    # Access render buffers for custom processing
    var size: Vector2i = render_scene_buffers.get_internal_size()
    # Custom rendering logic using RenderingDevice
    var rd: RenderingDevice = RenderingServer.get_rendering_device()
    # ... dispatch compute shaders or draw commands
```

> **Note:** CompositorEffect uses the low-level `RenderingDevice` API. This is an advanced feature — for most post-processing needs, use WorldEnvironment built-in effects or a SubViewport + shader approach.

### Built-in Compositor Uses

Godot uses the Compositor internally for:
- **SDFGI** (Signed Distance Field Global Illumination)
- **VoxelGI** probes
- **Screen-space reflections**

You can stack multiple CompositorEffects in order. Lower array indices execute first.

---

