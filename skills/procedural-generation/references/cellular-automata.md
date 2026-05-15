# Cellular Automata (Cave Generation)

Reference for `skills/procedural-generation/SKILL.md` — random fill + smoothing iterations for organic cave shapes. GDScript with TileMapLayer integration.

> ← Back to [SKILL.md](../SKILL.md)

---
## 4. Cellular Automata (Cave Generation)

Simulates natural-looking caves by iterating a simple rule: a cell becomes wall if most of its neighbors are walls.

### GDScript

```gdscript
class_name CaveGenerator
extends RefCounted

var rng := RandomNumberGenerator.new()

func generate(width: int, height: int, gen_seed: int, fill_chance: float = 0.45, iterations: int = 5) -> Array[Array]:
    rng.seed = gen_seed

    # Step 1: Random fill
    var grid: Array[Array] = []
    for y in height:
        var row: Array[bool] = []
        for x in width:
            # true = wall, false = floor
            var is_edge: bool = x == 0 or y == 0 or x == width - 1 or y == height - 1
            row.append(is_edge or rng.randf() < fill_chance)
        grid.append(row)

    # Step 2: Smooth with cellular automata rules
    for _i in iterations:
        grid = _smooth(grid, width, height)

    return grid

func _smooth(grid: Array[Array], width: int, height: int) -> Array[Array]:
    var new_grid: Array[Array] = []
    for y in height:
        var row: Array[bool] = []
        for x in width:
            var wall_count: int = _count_neighbors(grid, x, y, width, height)
            # Rule: become wall if 5+ of 9 cells (self + 8 neighbors) are walls
            row.append(wall_count >= 5)
        new_grid.append(row)
    return new_grid

func _count_neighbors(grid: Array[Array], cx: int, cy: int, width: int, height: int) -> int:
    var count: int = 0
    for dy in range(-1, 2):
        for dx in range(-1, 2):
            var nx: int = cx + dx
            var ny: int = cy + dy
            if nx < 0 or ny < 0 or nx >= width or ny >= height:
                count += 1  # out of bounds counts as wall
            elif grid[ny][nx]:
                count += 1
    return count
```

### Usage with TileMapLayer

```gdscript
func apply_cave_to_tilemap(tile_map: TileMapLayer, grid: Array[Array]) -> void:
    var wall_tile := Vector2i(0, 0)
    var floor_tile := Vector2i(1, 0)

    for y in grid.size():
        for x in grid[y].size():
            var tile := wall_tile if grid[y][x] else floor_tile
            tile_map.set_cell(Vector2i(x, y), 0, tile)
```

