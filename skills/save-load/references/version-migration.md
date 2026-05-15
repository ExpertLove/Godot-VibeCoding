# Version Migration

Reference for `skills/save-load/SKILL.md` — incremental migration from older save versions on load.

> ← Back to [SKILL.md](../SKILL.md)

---
## 6. Version Migration

Always store a `version` integer in every save file. Apply migrations incrementally so any old save can be brought forward to the current format regardless of how many versions it has missed.

```gdscript
func _migrate(data: Dictionary) -> Dictionary:
	var version: int = data.get("version", 0)

	if version < 1:
		# v0 → v1: inventory did not exist, add empty array
		data["player"]["inventory"] = []
		version = 1

	if version < 2:
		# v1 → v2: skills system added, seed from empty array
		data["player"]["skills"] = []
		version = 2

	# v2 → v3: add stamina stat with default value
	if version < 3:
		data["player"]["stamina"] = 100
		version = 3

	data["version"] = CURRENT_VERSION
	return data
```

Key rules:
- Each migration block is additive — it only adds or transforms, never removes data
- Use `data.get("key", default)` defensively within migration blocks
- The version field must be written back before returning

---

