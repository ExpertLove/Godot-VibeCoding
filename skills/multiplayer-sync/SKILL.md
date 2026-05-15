---
name: multiplayer-sync
description: Use when synchronizing multiplayer state — MultiplayerSynchronizer, interpolation, prediction, and lag compensation
---

# Multiplayer Synchronization in Godot 4.3+

All examples target Godot 4.3+ with no deprecated APIs. 

> **Related skills:** **multiplayer-basics** for ENet setup, RPCs, and authority model, **dedicated-server** for headless export and deployment, **physics-system** for physics interpolation and RigidBody synchronization.

---

## 1. MultiplayerSynchronizer

`MultiplayerSynchronizer` is Godot's built-in node for replicating properties across the network. Add it as a child of the node whose state you want to share.

### What It Does

- Sends property values from the **authority** peer to all others at a configured interval
- Supports both **delta sync** (only changed values) and **full sync** (all values every tick)
- Allows **visibility filters** to control which peers receive updates

### Replication Config in the Editor

1. Select the `MultiplayerSynchronizer` node in the scene tree.
2. In the Inspector, open **Replication** and click **Add Property**.
3. Pick the parent node path and property name (e.g. `position`, `velocity`).
4. Set **Sync** (send every interval) or **Spawn** (send only on spawn) per property.
5. Set the **Replication Interval** (seconds). `0` means every physics frame.

### Key Properties

| Property | Description |
|---|---|
| `replication_interval` | Seconds between full sync updates. `0` = every physics frame |
| `delta_interval` | Seconds between delta sync updates. `0` = disabled |
| `public_visibility` | When `true`, updates go to all peers (default) |
| `visibility_filters` | Array of `Callable`s; each returns `true` if a peer should receive updates |

### Delta vs Full Sync

| Mode | How It Works | Best For |
|---|---|---|
| **Full sync** | Sends all configured properties every `replication_interval` | Simple objects, low property count |
| **Delta sync** | Sends only properties that changed since last sync, every `delta_interval` | Objects with many properties that change infrequently |

Use both together: set `replication_interval` for periodic full state and `delta_interval` for frequent change-only bursts.

### Visibility Filters (GDScript)

```gdscript
# Only send updates to peers within 500 units of this object.
func _ready() -> void:
    $MultiplayerSynchronizer.add_visibility_filter(_is_peer_in_range)

func _is_peer_in_range(peer_id: int) -> bool:
    var peer_player := _get_player_node(peer_id)
    if peer_player == null:
        return false
    return global_position.distance_to(peer_player.global_position) <= 500.0
```


## 2. Property Synchronization

### What to Sync

Sync the minimal state needed to reconstruct the visual on remote peers. Typical properties:

| Property | Type | Notes |
|---|---|---|
| `position` | `Vector2` / `Vector3` | Core transform — sync every frame or use interpolation |
| `velocity` | `Vector2` / `Vector3` | Helps remote prediction stay ahead of position snaps |
| `health` | `int` / `float` | Sync reliably on change; delta sync is ideal |
| `animation_state` | `String` / `int` | Sync on change; use an enum int to save bandwidth |
| `is_crouching` | `bool` | Low-change boolean; delta sync or RPC on change |

### Synced Player (GDScript)

```gdscript
# synced_player.gd
extends CharacterBody2D

## Sync interval in seconds — exposed so designers can tune per object type.
@export var sync_interval: float = 0.05  # 20 Hz

@export var speed: float = 200.0

# These properties are listed in the MultiplayerSynchronizer replication config.
var synced_position: Vector2 = Vector2.ZERO
var synced_velocity: Vector2 = Vector2.ZERO
var synced_health: int = 100
var synced_anim: int = 0  # 0 = idle, 1 = run, 2 = jump

@onready var _sync: MultiplayerSynchronizer = $MultiplayerSynchronizer


func _ready() -> void:
    _sync.replication_interval = sync_interval
    # Only the authority (owner) drives movement.
    set_physics_process(is_multiplayer_authority())


func _physics_process(_delta: float) -> void:
    # Authority: write canonical state so MultiplayerSynchronizer can replicate it.
    synced_position = global_position
    synced_velocity = velocity
    synced_anim     = _compute_anim_state()

## 3. Interpolation

`MultiplayerSynchronizer` updates target properties at sync intervals (e.g., 30 Hz), but rendering runs at frame rate (60+ Hz). Without interpolation, remote players appear to teleport between snapshots. The fix: store position snapshots with timestamps and lerp in `_process` toward the latest snapshot using a small offset (interpolation buffer ~100 ms).

> See [references/interpolation.md](references/interpolation.md) for the full GDScript interpolation buffer pattern (snapshot ring, latest-snapshot interpolation, render-time lerp).

---

## 4. Client-Side Prediction

For local-player responsiveness: predict movement immediately on client, send input to server, reconcile when server snapshot arrives. If server diverges from client prediction beyond a threshold, snap; otherwise smoothly lerp the correction over 100-200 ms.

> See [references/client-prediction.md](references/client-prediction.md) for the full predict-and-reconcile pattern (input ring buffer, server reconciliation, replay) in GDScript.

---

## 5. Lag Compensation

For hit-scan weapons in fast-paced games: when the server validates a hit, it rewinds the world state to the client's view-time (`now - client_rtt/2 - interp_delay`) and tests the hit against that historical state.

> See [references/lag-compensation.md](references/lag-compensation.md) for the snapshot-history pattern, view-time calculation, and a hit-scan validator in GDScript.

---

## 6. State vs Input Synchronization

Choose the synchronization model that fits your game's needs:

| Factor | Sync State | Sync Inputs |
|---|---|---|
| **What is sent** | Current property values (position, health, etc.) | Player input actions each frame |
| **Who simulates** | Authority only; others receive results | All peers run the same simulation |
| **Determinism required** | No | Yes — every peer must produce identical output from the same inputs |
| **Bandwidth** | Higher — full state sent each interval | Lower — small input structs per frame |
| **Responsiveness** | Lower — non-authority peers wait for next sync tick | Higher — local prediction is trivial when deterministic |
| **Complexity** | Lower — no reconciliation loop | Higher — requires deterministic physics, fixed-point math, or lockstep |
| **Best for** | Action games, shooters, most real-time games | Fighting games, RTS, turn-based, simulation games |
| **Lag compensation needed** | Yes, for hit detection | Usually not — all peers are in sync |

**Hybrid approach** (most real-time games): sync inputs for the local player's character (enabling prediction), sync state for all other objects and game events.

---

## 7. Bandwidth Optimization

Four levers: **sync only changed properties** (replication-config flag per property), **quantize floats** (Vector3 components in mm not floats — 16-bit cuts bytes by 2×), **distance-based sync rate** (far-away objects sync at 5 Hz, close at 30 Hz), and **channel selection** (reliable for state changes that must arrive, unreliable for position streams that get superseded).

> See [references/bandwidth-optimization.md](references/bandwidth-optimization.md) for full GDScript recipes for each lever, plus the reliable-vs-unreliable channel decision tree.

---

## 8. Implementation Checklist

- [ ] `MultiplayerSynchronizer` is a direct child of the node it replicates
- [ ] Only the **authority** peer writes to synced properties; others are read-only
- [ ] `set_multiplayer_authority()` is called at spawn time with the correct peer ID
- [ ] `replication_interval` and `delta_interval` are tuned for the object's update rate
- [ ] Remote player visuals use interpolation in `_process`, not `_physics_process`
- [ ] Interpolation stores previous and current state and blends using `Engine.get_physics_interpolation_fraction()`
- [ ] Client-side prediction is applied only to the local player's own character
- [ ] Pending input buffer is bounded (max ~128 ticks) to prevent memory growth
- [ ] Reconciliation threshold prevents jitter from micro-corrections
- [ ] Position and velocity use `unreliable` RPC; state changes use `reliable`
- [ ] Float quantization is applied before sending position data over the network
- [ ] Lag compensation snapshot history is pruned each tick to a bounded window
- [ ] Server validates all hit detection; clients never self-report kills
- [ ] Distance-based sync rate reduces bandwidth for far-away objects
