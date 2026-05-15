# Action Rebinding at Runtime

Reference for `skills/input-handling/SKILL.md` — full action-rebinding flow with GDScript: capture new key, swap action's events, persist via ConfigFile, restore on launch.

> ← Back to [SKILL.md](../SKILL.md)

---
## 7. Action Rebinding at Runtime

Allow players to change their key bindings in-game.

### GDScript

```gdscript
# rebind_button.gd — attach to a Button in a settings menu
extends Button

@export var action_name: String = "jump"

var _is_listening: bool = false


func _ready() -> void:
    _update_label()


func _pressed() -> void:
    _is_listening = true
    text = "Press a key..."


func _unhandled_input(event: InputEvent) -> void:
    if not _is_listening:
        return

    # Accept keyboard, mouse button, and gamepad button events
    if not (event is InputEventKey or event is InputEventMouseButton or event is InputEventJoypadButton):
        return

    # Ignore modifier-only presses (Shift, Ctrl, Alt alone)
    if event is InputEventKey and event.keycode in [KEY_SHIFT, KEY_CTRL, KEY_ALT, KEY_META]:
        return

    # Replace all existing events for this action
    InputMap.action_erase_events(action_name)
    InputMap.action_add_event(action_name, event)

    _is_listening = false
    _update_label()
    get_viewport().set_input_as_handled()


func _update_label() -> void:
    var events := InputMap.action_get_events(action_name)
    if events.size() > 0:
        text = "%s: %s" % [action_name, events[0].as_text()]
    else:
        text = "%s: (unbound)" % action_name
```


### Saving & Loading Bindings

```gdscript
# Save current bindings to ConfigFile
func save_bindings(config: ConfigFile) -> void:
    for action in InputMap.get_actions():
        # Skip built-in ui_* actions
        if action.begins_with("ui_"):
            continue
        var events := InputMap.action_get_events(action)
        var event_data: Array[Dictionary] = []
        for event in events:
            event_data.append({
                "type": event.get_class(),
                "data": var_to_str(event)
            })
        config.set_value("input", action, event_data)
    config.save("user://input_bindings.cfg")


# Load saved bindings
func load_bindings() -> void:
    var config := ConfigFile.new()
    if config.load("user://input_bindings.cfg") != OK:
        return
    for action in config.get_section_keys("input"):
        if not InputMap.has_action(action):
            continue
        InputMap.action_erase_events(action)
        var event_data: Array = config.get_value("input", action, [])
        for entry in event_data:
            var event: InputEvent = str_to_var(entry["data"])
            if event:
                InputMap.action_add_event(action, event)
```



---

