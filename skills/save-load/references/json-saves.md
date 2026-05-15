# JSON — Game Saves

Reference for `skills/save-load/SKILL.md` — `JSON.stringify` / `JSON.parse_string` for game save state. full GDScript implementation.

> ← Back to [SKILL.md](../SKILL.md)

---
## 3. JSON — Game Saves

Use JSON for game saves. It is portable, debuggable, and easy to version-migrate.

### GDScript

```gdscript
# save_manager.gd — add as autoload named SaveManager
extends Node

const SAVE_DIR       := "user://saves/"
const SAVE_EXTENSION := ".json"
const CURRENT_VERSION := 2


func _ready() -> void:
	DirAccess.make_dir_recursive_absolute(SAVE_DIR)


# ── Save ──────────────────────────────────────────────────────────────────────

func save_game(slot_name: String) -> bool:
	var player := get_tree().get_first_node_in_group("player")
	var world  := get_tree().get_first_node_in_group("world")

	var data: Dictionary = {
		"version":   CURRENT_VERSION,
		"timestamp": Time.get_unix_time_from_system(),
		"player":    _serialize_player(player),
		"world":     _serialize_world(world),
	}

	var json_string := JSON.stringify(data, "\t")
	var path        := SAVE_DIR + slot_name + SAVE_EXTENSION
	var file        := FileAccess.open(path, FileAccess.WRITE)
	if file == null:
		push_error("SaveManager: cannot open '%s' for writing — error %d" % [path, FileAccess.get_open_error()])
		return false

	file.store_string(json_string)
	return true


func _serialize_player(player: Node) -> Dictionary:
	return {
		"position":  {"x": player.global_position.x, "y": player.global_position.y},
		"health":    player.health,
		"inventory": player.inventory.duplicate(),
	}


func _serialize_world(world: Node) -> Dictionary:
	var enemies: Array = []
	for enemy in get_tree().get_nodes_in_group("enemies"):
		enemies.append({
			"scene_path": enemy.scene_file_path,
			"position":   {"x": enemy.global_position.x, "y": enemy.global_position.y},
			"health":     enemy.health,
		})
	return {"enemies": enemies}


# ── Load ──────────────────────────────────────────────────────────────────────

func load_game(slot_name: String) -> bool:
	var path := SAVE_DIR + slot_name + SAVE_EXTENSION
	if not FileAccess.file_exists(path):
		push_error("SaveManager: save file not found at '%s'" % path)
		return false

	var file := FileAccess.open(path, FileAccess.READ)
	if file == null:
		push_error("SaveManager: cannot open '%s' for reading — error %d" % [path, FileAccess.get_open_error()])
		return false

	var json   := JSON.new()
	var err    := json.parse(file.get_as_text())
	if err != OK:
		push_error("SaveManager: JSON parse error in '%s': %s" % [path, json.get_error_message()])
		return false

	var data: Dictionary = json.data
	data = _migrate(data)

	var player := get_tree().get_first_node_in_group("player")
	var world  := get_tree().get_first_node_in_group("world")
	_deserialize_player(player, data["player"])
	_deserialize_world(world, data["world"])
	return true


func _deserialize_player(player: Node, data: Dictionary) -> void:
	player.global_position = Vector2(data["position"]["x"], data["position"]["y"])
	player.health          = data["health"]
	player.inventory       = data["inventory"].duplicate()


func _deserialize_world(world: Node, data: Dictionary) -> void:
	# Remove existing enemies spawned at runtime
	for enemy in get_tree().get_nodes_in_group("enemies"):
		enemy.queue_free()

	for entry: Dictionary in data["enemies"]:
		var scene: PackedScene = load(entry["scene_path"])
		if scene == null:
			push_error("SaveManager: missing scene '%s'" % entry["scene_path"])
			continue
		var enemy: Node = scene.instantiate()
		world.add_child(enemy)
		enemy.global_position = Vector2(entry["position"]["x"], entry["position"]["y"])
		enemy.health          = entry["health"]


# ── Helpers ───────────────────────────────────────────────────────────────────

func get_save_slots() -> Array[String]:
	var slots: Array[String] = []
	var dir := DirAccess.open(SAVE_DIR)
	if dir == null:
		return slots
	dir.list_dir_begin()
	var file_name := dir.get_next()
	while file_name != "":
		if not dir.current_is_dir() and file_name.ends_with(SAVE_EXTENSION):
			slots.append(file_name.trim_suffix(SAVE_EXTENSION))
		file_name = dir.get_next()
	return slots


func delete_save(slot_name: String) -> bool:
	var path := SAVE_DIR + slot_name + SAVE_EXTENSION
	var err  := DirAccess.remove_absolute(path)
	if err != OK:
		push_error("SaveManager: failed to delete '%s' — error %d" % [path, err])
		return false
	return true


# ── Migration ─────────────────────────────────────────────────────────────────

func _migrate(data: Dictionary) -> Dictionary:
	var version: int = data.get("version", 0)

	if version < 1:
		# v0 → v1: add inventory array
		data["player"]["inventory"] = []
		version = 1

	if version < 2:
		# v1 → v2: add skills array to player
		data["player"]["skills"] = []
		version = 2

	data["version"] = CURRENT_VERSION
	return data
```

