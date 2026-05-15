# Gizmos Deep Dive

Reference for `skills/addon-development/SKILL.md` — `EditorNode3DGizmoPlugin` subclass with `_init`, `_get_gizmo_name`, `_has_gizmo`, `_redraw`, `_get_handle_value`, `_set_handle`, `_commit_handle`. 

> ← Back to [SKILL.md](../SKILL.md)

---

## 7. Gizmos

`EditorNode3DGizmoPlugin` adds interactive handles and visual overlays to 3D nodes in the viewport. Register the plugin from your `EditorPlugin`.

### GDScript

```gdscript
# my_gizmo_plugin.gd
@tool
extends EditorNode3DGizmoPlugin

const HANDLE_RADIUS := 0.15


func _init() -> void:
    # Create a named material for the gizmo lines/handles.
    create_material("main", Color(0.5, 1.0, 0.0))
    create_handle_material("handles")


# Displayed in the View menu under "Show Gizmos".
func _get_gizmo_name() -> String:
    return "MyNode3DGizmo"


# Return true if this plugin should draw a gizmo for the given node.
func _has_gizmo(node: Node3D) -> bool:
    return node is MyNode3D


# Called whenever the node changes or the viewport is redrawn.
# Re-add all lines and handles here — do not cache between calls.
func _redraw(gizmo: EditorNode3DGizmo) -> void:
    gizmo.clear()

    var node := gizmo.get_node_3d() as MyNode3D
    if node == null:
        return

    # Draw a line from the node origin to a target point.
    var lines := PackedVector3Array([Vector3.ZERO, node.target_offset])
    gizmo.add_lines(lines, get_material("main", gizmo), false)

    # Add a draggable handle at the target offset position.
    var handles := PackedVector3Array([node.target_offset])
    gizmo.add_handles(handles, get_material("handles", gizmo), [])


# Return the current value of a handle as a Transform3D or Vector3
# so the editor can restore it on undo.
func _get_handle_value(gizmo: EditorNode3DGizmo, handle_id: int, secondary: bool) -> Variant:
    return (gizmo.get_node_3d() as MyNode3D).target_offset


# Called while dragging a handle. camera is the current viewport camera.
# point is the screen-space cursor position.
func _set_handle(
    gizmo: EditorNode3DGizmo,
    handle_id: int,
    secondary: bool,
    camera: Camera3D,
    point: Vector2
) -> void:
    var node := gizmo.get_node_3d() as MyNode3D
    # Project the screen point onto the XZ plane at the node's Y position.
    var from := camera.project_ray_origin(point)
    var dir  := camera.project_ray_normal(point)
    var dist := (node.global_position.y - from.y) / dir.y
    node.target_offset = from + dir * dist - node.global_position
    # Redraw after every drag update.
    _redraw(gizmo)


# Restore the handle to the value saved by _get_handle_value (for undo/redo).
func _commit_handle(
    gizmo: EditorNode3DGizmo,
    handle_id: int,
    secondary: bool,
    restore: Variant,
    cancel: bool
) -> void:
    var node := gizmo.get_node_3d() as MyNode3D
    if cancel:
        node.target_offset = restore
    else:
        # Register with undo/redo so Ctrl+Z works.
        get_undo_redo().create_action("Move MyNode3D Handle")
        get_undo_redo().add_do_property(node, "target_offset", node.target_offset)
        get_undo_redo().add_undo_property(node, "target_offset", restore)
        get_undo_redo().commit_action()
```

Register from `EditorPlugin`:

```gdscript
# plugin.gd
var _gizmo_plugin: EditorNode3DGizmoPlugin

func _enter_tree() -> void:
    _gizmo_plugin = preload("res://addons/my_plugin/my_gizmo_plugin.gd").new()
    add_node_3d_gizmo_plugin(_gizmo_plugin)

func _exit_tree() -> void:
    remove_node_3d_gizmo_plugin(_gizmo_plugin)
```


---

