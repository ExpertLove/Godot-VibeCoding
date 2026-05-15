# Branching and Conditions

Reference for `skills/dialogue-system/SKILL.md` — choice-driven branching, condition expressions on lines, condition evaluator using `Expression`. GDScript.

> ← Back to [SKILL.md](../SKILL.md)

---
## 5. Branching and Conditions

### Choice Nodes

A `DialogueLine` with a non-empty `choices` array acts as a branch point. Each choice entry is a plain Dictionary:

```gdscript
# Inside a DialogueLine resource (set in code or loaded from JSON)
var ask_line := DialogueLine.new()
ask_line.speaker = "Guard"
ask_line.text    = "What brings you here, traveller?"
ask_line.choices = [
    {"text": "I seek the king.",       "next_line_id": "seek_king"},
    {"text": "Just passing through.",  "next_line_id": "passing_through"},
    {"text": "I have a letter.",       "next_line_id": "letter_branch",
     "condition": "GameState.has_item('royal_letter')"},
]
```

The third choice only appears when `GameState.has_item('royal_letter')` is true. `DialogueManager._visible_choices()` filters the list before emitting `choice_presented`.

### Condition Evaluator

`DialogueManager._evaluate_condition()` uses Godot's built-in `Expression` class, which can call methods on any object passed as the base instance. Wire it to a `GameState` autoload for clean condition strings:

```gdscript
# game_state.gd — autoload named GameState
extends Node

var flags:     Dictionary = {}  # arbitrary boolean flags
var inventory: Inventory         # set by the player scene


func has_flag(key: String) -> bool:
    return flags.get(key, false)


func set_flag(key: String, value: bool = true) -> void:
    flags[key] = value


func has_item(item_id: String, quantity: int = 1) -> bool:
    if inventory == null:
        return false
    var item: ItemData = ItemRegistry.get_item(item_id)
    return item != null and inventory.has_item(item, quantity)


func quest_stage(quest_id: String) -> int:
    return flags.get("quest_%s_stage" % quest_id, 0)
```

Pass `GameState` as the expression base to resolve method calls:

```gdscript
# In DialogueManager._evaluate_condition():
var result = expr.execute([], GameState)   # ← pass autoload as base instance
```

Condition strings in dialogue data then read naturally:

```
"GameState.has_flag('met_queen')"
"GameState.quest_stage('main') >= 2"
"GameState.has_item('potion', 3)"
```

---

