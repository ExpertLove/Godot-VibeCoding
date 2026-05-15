# Wave Function Collapse (WFC)

Reference for `skills/procedural-generation/SKILL.md` — tile-set constraint solver. Concept overview and simplified algorithm in GDScript.

> ← Back to [SKILL.md](../SKILL.md)

---
## 5. Wave Function Collapse (WFC) — Concept

WFC generates patterns by collapsing tile possibilities based on adjacency constraints. It produces visually coherent results from a small set of rules.

### Core Algorithm (Simplified)

```gdscript
class_name SimpleWFC
extends RefCounted

# Each cell holds a set of possible tile indices
var grid: Array[Array] = []   # grid[y][x] = Array[int] (possible tiles)
var rules: Dictionary = {}     # rules[tile_id] = {"up": [...], "down": [...], "left": [...], "right": [...]}
var rng := RandomNumberGenerator.new()

func setup(width: int, height: int, tile_count: int, gen_seed: int) -> void:
    rng.seed = gen_seed
    grid.clear()
    for y in height:
        var row: Array[Array] = []
        for x in width:
            var possibilities: Array[int] = []
            for t in tile_count:
                possibilities.append(t)
            row.append(possibilities)
        grid.append(row)

func collapse() -> bool:
    while true:
        # Find cell with fewest possibilities (lowest entropy)
        var min_cell := Vector2i(-1, -1)
        var min_count := 999
        for y in grid.size():
            for x in grid[y].size():
                var count: int = grid[y][x].size()
                if count > 1 and count < min_count:
                    min_count = count
                    min_cell = Vector2i(x, y)

        if min_cell == Vector2i(-1, -1):
            return true  # all collapsed — success

        # Collapse: pick a random possibility
        var cell: Array[int] = grid[min_cell.y][min_cell.x]
        if cell.is_empty():
            return false  # contradiction — no valid tiles
        var chosen: int = cell[rng.randi() % cell.size()]
        grid[min_cell.y][min_cell.x] = [chosen]

        # Propagate constraints to neighbors
        _propagate(min_cell)

    return true

func _propagate(pos: Vector2i) -> void:
    var stack: Array[Vector2i] = [pos]
    while not stack.is_empty():
        var current: Vector2i = stack.pop_back()
        var current_tiles: Array[int] = grid[current.y][current.x]

        for dir in [Vector2i(0, -1), Vector2i(0, 1), Vector2i(-1, 0), Vector2i(1, 0)]:
            var neighbor: Vector2i = current + dir
            if neighbor.x < 0 or neighbor.y < 0 or neighbor.y >= grid.size() or neighbor.x >= grid[0].size():
                continue

            var dir_name: String = _dir_to_name(dir)
            var allowed: Array[int] = []
            for tile in current_tiles:
                if rules.has(tile) and rules[tile].has(dir_name):
                    for allowed_tile in rules[tile][dir_name]:
                        if allowed_tile not in allowed:
                            allowed.append(allowed_tile)

            var neighbor_tiles: Array[int] = grid[neighbor.y][neighbor.x]
            var new_tiles: Array[int] = neighbor_tiles.filter(func(t: int) -> bool: return t in allowed)

            if new_tiles.size() < neighbor_tiles.size():
                grid[neighbor.y][neighbor.x] = new_tiles
                stack.append(neighbor)

func _dir_to_name(dir: Vector2i) -> String:
    if dir == Vector2i(0, -1): return "up"
    if dir == Vector2i(0, 1):  return "down"
    if dir == Vector2i(-1, 0): return "left"
    return "right"
```


> **For production WFC**, consider the community addon [godot-wfc](https://github.com/AlexeyBond/godot-wfc) which provides editor integration, TileMap support, and 3D grid WFC.

---

