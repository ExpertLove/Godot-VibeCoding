# Client-Side Prediction

Reference for `skills/multiplayer-sync/SKILL.md` — predict-and-reconcile pattern: client predicts locally, server is authoritative, reconcile on snapshot diff. Full GDScript implementation.

> ← Back to [SKILL.md](../SKILL.md)

---
## 4. Client-Side Prediction

Client-side prediction lets the local player's input feel instant by applying it locally before the server confirms it. When the server sends a correction, the client reconciles by replaying any unconfirmed inputs on top of the corrected state.

### Pattern (GDScript)

```gdscript
# predicted_player.gd — authority is the server; this runs on the local client
extends CharacterBody2D

@export var speed: float = 200.0

# Ring buffer of unacknowledged inputs indexed by tick number.
var _pending_inputs: Dictionary = {}
var _current_tick: int = 0

# Last server-confirmed state.
var _server_position: Vector2 = Vector2.ZERO
var _server_tick: int = -1


func _physics_process(delta: float) -> void:
    var input_dir: Vector2 = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")

    # 1. Predict: apply input locally right now.
    _apply_input(input_dir, delta)

    # 2. Store input so we can replay it if the server corrects us.
    _pending_inputs[_current_tick] = {"dir": input_dir, "delta": delta}
    _current_tick += 1

    # 3. Send input to server every frame (or batch at a lower rate).
    if multiplayer.is_server():
        return
    _send_input_to_server.rpc_id(1, input_dir, _current_tick - 1)


func _apply_input(dir: Vector2, delta: float) -> void:
    velocity = dir * speed
    move_and_slide()


@rpc("authority", "call_remote", "unreliable")
func _receive_server_correction(corrected_pos: Vector2, ack_tick: int) -> void:
    # Server has confirmed state up to ack_tick. Snap and replay.
    _server_position = corrected_pos
    _server_tick     = ack_tick

    # Drop all inputs the server has already processed.
    for tick in _pending_inputs.keys():
        if tick <= ack_tick:
            _pending_inputs.erase(tick)

    # Reconcile: rewind to server position then replay pending inputs.
    global_position = _server_position
    for tick in _pending_inputs.keys():
        var inp: Dictionary = _pending_inputs[tick]
        _apply_input(inp["dir"], inp["delta"])


@rpc("any_peer", "call_remote", "unreliable")
func _send_input_to_server(dir: Vector2, tick: int) -> void:
    # Server receives client input, simulates, then confirms.
    _apply_input(dir, get_physics_process_delta_time())
    _receive_server_correction.rpc_id(
        multiplayer.get_remote_sender_id(),
        global_position,
        tick
    )
```

**Key points:**
- Keep `_pending_inputs` bounded — evict entries older than a reasonable window (e.g. 128 ticks).
- Only reconcile if the corrected position differs from the predicted position by more than a threshold (e.g. 2 px / 0.05 m) to avoid jitter on micro-corrections.
- Use `unreliable` channel for position corrections — timeliness matters more than ordering.

---

