# @export Node Injection

Reference for `skills/dependency-injection/SKILL.md` — exposing dependencies as `@export` properties for Inspector wiring. GDScript.

> ← Back to [SKILL.md](../SKILL.md)

---
## 4. @export Node Injection

The most Godot-idiomatic DI pattern. Declare a dependency as `@export` and wire it in the editor Inspector — or assign it from a parent in code. The node never needs to know *where* its dependency lives in the tree.

This is preferred over `get_node("../../SomeNode")` hard-coded paths, which break silently when the scene tree is reorganised.

### GDScript

```gdscript
# health_component.gd — receives its audio dependency via @export
class_name HealthComponent
extends Node

@export var audio: AudioManager          ## Set in the Inspector or by parent
@export var max_health: int = 100

var current_health: int


func _ready() -> void:
    current_health = max_health


func take_damage(amount: int) -> void:
    current_health = clampi(current_health - amount, 0, max_health)
    # Uses the injected reference — no direct autoload access
    if audio != null:
        audio.play_sfx("hurt")
    health_changed.emit(current_health, max_health)
    if current_health == 0:
        died.emit()


signal health_changed(current: int, maximum: int)
signal died
```

```gdscript
# player.gd — wires the component in the editor or in _ready()
extends CharacterBody2D

@export var health: HealthComponent      ## Drag HealthComponent node here in Inspector
```




---

