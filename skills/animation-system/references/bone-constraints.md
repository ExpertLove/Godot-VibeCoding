# BoneConstraint3D Modifiers

Reference for `skills/animation-system/SKILL.md` — Godot 4.5+ `BoneConstraint3D` subclasses (Aim / Copy / Convert) for bone-relative skeleton modifiers.

> ← Back to [SKILL.md](../SKILL.md)

---

## BoneConstraint3D Modifiers (Godot 4.5+)

Godot 4.5 introduces `BoneConstraint3D` — a new base class for skeleton modifiers that operate relative to another bone rather than a world-space target. Three concrete subclasses ship with 4.5:

| Modifier | What it does |
|----------|-------------|
| `AimModifier3D` | Rotates a bone to aim along its primary axis toward a reference bone |
| `CopyTransformModifier3D` | Copies position/rotation/scale from one bone to another (useful for mirroring or binding secondary rigs) |
| `ConvertTransformModifier3D` | Converts between transform spaces — translates, rotates, or scales a bone based on another bone's transform, with remapping |

These complement `LookAtModifier3D` (which targets a world-space `Node3D`). Use `AimModifier3D` when the aim target is itself a bone on the same skeleton.

**Scene structure:**

```
Character (CharacterBody3D)
└── Skeleton3D
    ├── AimModifier3D         ← child of Skeleton3D
    └── CopyTransformModifier3D
```

**AimModifier3D — bone-to-bone aiming:**

```gdscript
@onready var skeleton: Skeleton3D = $Skeleton3D

func _ready() -> void:
    var aim := AimModifier3D.new()
    skeleton.add_child(aim)
    aim.bone_name = "RightArm"           # bone that aims
    aim.target_bone_name = "RightHand"   # bone it aims toward
    aim.primary_rotation_axis = Vector3.RIGHT  # axis to rotate around

    # Limit how far the bone can rotate
    aim.use_angle_limitation = true
    aim.symmetry_limitation = deg_to_rad(90.0)
```



**CopyTransformModifier3D — mirror/bind bones:**

```gdscript
func _ready() -> void:
    var copy := CopyTransformModifier3D.new()
    skeleton.add_child(copy)
    copy.bone_name = "LeftArm"        # bone receiving the transform
    copy.source_bone_name = "RightArm"  # bone being copied
    copy.copy_position = false
    copy.copy_rotation = true
    copy.copy_scale = false
```



> **Note:** API property names were finalized at Godot 4.5 release. If property names differ in your Godot version, verify via the built-in Inspector on the modifier node. See [PR #100984](https://github.com/godotengine/godot/pull/100984) for the authoritative property list.

> **When to use:** Prefer `BoneConstraint3D` subclasses over manual bone transform manipulation in `_process()` — they integrate with the modifier pipeline and respect the animation blend stack.
