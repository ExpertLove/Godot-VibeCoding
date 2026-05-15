# Dialogue UI

Reference for `skills/dialogue-system/SKILL.md` — scene tree (CanvasLayer + RichTextLabel + choices VBoxContainer), typewriter text, choice button spawning, signal wiring. GDScript.

> ← Back to [SKILL.md](../SKILL.md)

---
## 6. Dialogue UI

### Scene Structure

```
DialogueUI (Control)
  ├─ PanelContainer
  │    ├─ HBoxContainer
  │    │    ├─ TextureRect      (portrait)
  │    │    └─ VBoxContainer
  │    │         ├─ Label       (speaker_name)
  │    │         └─ RichTextLabel (dialogue_text)
  │    └─ VBoxContainer         (choice_container)
  │         └─ Button × N       (instantiated at runtime)
  └─ Timer                      (typewriter_timer)
```

### GDScript

```gdscript
# dialogue_ui.gd
class_name DialogueUI
extends Control

@export var manager: DialogueManager  # assign the autoload or a node ref

@onready var speaker_label:    Label          = $PanelContainer/HBoxContainer/VBoxContainer/Label
@onready var dialogue_text:    RichTextLabel  = $PanelContainer/HBoxContainer/VBoxContainer/RichTextLabel
@onready var portrait:         TextureRect    = $PanelContainer/HBoxContainer/TextureRect
@onready var choice_container: VBoxContainer  = $PanelContainer/VBoxContainer
@onready var typewriter_timer: Timer          = $Timer

const TYPEWRITER_INTERVAL := 0.04  # seconds per character


func _ready() -> void:
    if manager == null:
        manager = get_node("/root/DialogueManager")
    manager.line_displayed.connect(_on_line_displayed)
    manager.choice_presented.connect(_on_choice_presented)
    manager.dialogue_ended.connect(_on_dialogue_ended)
    hide()


# ── Input ─────────────────────────────────────────────────────────────────────

func _unhandled_input(event: InputEvent) -> void:
    if not visible:
        return
    if event.is_action_pressed("ui_accept"):
        if typewriter_timer.is_stopped():
            manager.advance()
        else:
            # Skip typewriter — reveal full text immediately
            typewriter_timer.stop()
            dialogue_text.visible_characters = -1


# ── Signal handlers ───────────────────────────────────────────────────────────

func _on_line_displayed(line: DialogueLine) -> void:
    show()
    _clear_choices()

    speaker_label.text = line.speaker
    # Variable interpolation happens before display (see section 8)
    dialogue_text.text = _interpolate(line.text)
    dialogue_text.visible_characters = 0

    # Optionally set portrait from a Dictionary keyed by speaker name
    # portrait.texture = PortraitRegistry.get_portrait(line.speaker)

    typewriter_timer.wait_time = TYPEWRITER_INTERVAL
    typewriter_timer.start()


func _on_typewriter_tick() -> void:
    if dialogue_text.visible_characters < dialogue_text.get_total_character_count():
        dialogue_text.visible_characters += 1
    else:
        typewriter_timer.stop()


func _on_choice_presented(choices: Array) -> void:
    _clear_choices()
    for i in choices.size():
        var choice: Dictionary = choices[i]
        var btn := Button.new()
        btn.text = choice.get("text", "")
        btn.pressed.connect(manager.choose.bind(i))
        choice_container.add_child(btn)


func _on_dialogue_ended() -> void:
    hide()
    _clear_choices()


# ── Helpers ───────────────────────────────────────────────────────────────────

func _clear_choices() -> void:
    for child in choice_container.get_children():
        child.queue_free()


# Variable interpolation — see section 8.
func _interpolate(text: String) -> String:
    return text.format({
        "player_name": GameState.get("player_name") if GameState.get("player_name") else "Hero",
    })
```

Connect the `Timer`'s `timeout` signal to `_on_typewriter_tick` in the editor or in `_ready()`:

```gdscript
typewriter_timer.timeout.connect(_on_typewriter_tick)
```

---

