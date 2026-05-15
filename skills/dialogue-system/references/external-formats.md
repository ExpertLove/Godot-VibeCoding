# External Dialogue Formats

Reference for `skills/dialogue-system/SKILL.md` — loading dialogue from JSON, integration notes for the Dialogic addon.

> ← Back to [SKILL.md](../SKILL.md)

---
## 7. External Formats

### JSON Dialogue Files

Storing dialogue as plain JSON decouples writing from the Godot editor and lets writers use any text editor or spreadsheet tool. Load at runtime with `FileAccess`:

```gdscript
# dialogue_loader.gd
class_name DialogueLoader
extends RefCounted

static func load_from_json(path: String) -> DialogueData:
    var file := FileAccess.open(path, FileAccess.READ)
    if file == null:
        push_error("DialogueLoader: cannot open '%s'" % path)
        return null

    var json   := JSON.new()
    var err    := json.parse(file.get_as_text())
    if err != OK:
        push_error("DialogueLoader: JSON parse error in '%s' — %s" % [path, json.get_error_message()])
        return null

    var raw: Dictionary = json.data
    var data := DialogueData.new()
    data.start_line_id = raw.get("start_line_id", "")

    for id: String in raw.get("lines", {}).keys():
        var entry: Dictionary = raw["lines"][id]
        var line              := DialogueLine.new()
        line.speaker          = entry.get("speaker", "")
        line.text             = entry.get("text", "")
        line.choices          = entry.get("choices", [])
        line.next_line_id     = entry.get("next_line_id", "")
        line.condition        = entry.get("condition", "")
        data.lines[id]        = line

    return data
```

Example JSON layout (`res://dialogue/guard.json`):

```json
{
  "start_line_id": "greet",
  "lines": {
    "greet": {
      "speaker": "Guard",
      "text": "Halt! State your business.",
      "choices": [
        { "text": "I bring a message.",  "next_line_id": "message" },
        { "text": "Never mind.",         "next_line_id": "" }
      ]
    },
    "message": {
      "speaker": "Guard",
      "text": "Very well. You may pass.",
      "next_line_id": ""
    }
  }
}
```

### Dialogic Addon

For larger projects, **[Dialogic](https://github.com/coppolaemilio/dialogic)** is the most widely used Godot dialogue addon. It provides a visual timeline editor, character management, portrait handling, and its own condition/event system. Consider Dialogic when:

- The writing team is non-technical and needs a visual editor.
- You need built-in localization, save/load of dialogue state, or audio cue events.
- The dialogue graph is large enough that manual JSON management becomes error-prone.

For small-to-medium projects the hand-rolled system in this skill keeps dependencies minimal and stays fully under your control.

---

