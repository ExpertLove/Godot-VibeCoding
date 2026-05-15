# Lobby Management

Reference for `skills/dedicated-server/SKILL.md` — full lobby system: per-player state dictionary, max-players cap, ready-toggle RPC, GDScript.

> ← Back to [SKILL.md](../SKILL.md)

---
## 3. Lobby System

### GDScript

```gdscript
# lobby_manager.gd — autoload named LobbyManager, runs on server only
extends Node

## Maximum concurrent players. Set via --max-players CLI argument.
var max_players: int = 8

## player_list[peer_id] = { "username": String, "ready": bool }
var player_list: Dictionary = {}

signal player_joined(peer_id: int)
signal player_left(peer_id: int)
signal all_players_ready()


func _ready() -> void:
    if not multiplayer.is_server():
        return
    multiplayer.peer_connected.connect(_on_peer_connected)
    multiplayer.peer_disconnected.connect(_on_peer_disconnected)


# ── Connection handling ───────────────────────────────────────────────────────

func _on_peer_connected(peer_id: int) -> void:
    if player_list.size() >= max_players:
        # Reject: lobby is full.
        _kick_peer.rpc_id(peer_id, "Lobby is full")
        return

    player_list[peer_id] = {"username": "Player%d" % peer_id, "ready": false}
    print("[Lobby] Peer %d joined (%d/%d)" % [peer_id, player_list.size(), max_players])

    # Send the new player the current lobby state.
    _sync_lobby_state.rpc_id(peer_id, player_list)
    # Notify everyone else.
    _notify_player_joined.rpc(peer_id, player_list[peer_id]["username"])
    player_joined.emit(peer_id)


func _on_peer_disconnected(peer_id: int) -> void:
    if not player_list.has(peer_id):
        return
    var username: String = player_list[peer_id]["username"]
    player_list.erase(peer_id)
    _notify_player_left.rpc(peer_id, username)
    print("[Lobby] Peer %d left (%d/%d)" % [peer_id, player_list.size(), max_players])
    player_left.emit(peer_id)


# ── Ready state ───────────────────────────────────────────────────────────────

## Clients call this RPC to toggle their ready state.
@rpc("any_peer", "reliable")
func set_ready(is_ready: bool) -> void:
    var sender_id := multiplayer.get_remote_sender_id()
    if not player_list.has(sender_id):
        return
    player_list[sender_id]["ready"] = is_ready
    _notify_ready_changed.rpc(sender_id, is_ready)
    _check_all_ready()


func _check_all_ready() -> void:
    if player_list.is_empty():
        return
    for data in player_list.values():
        if not data["ready"]:
            return
    print("[Lobby] All players ready — starting game")
    all_players_ready.emit()
    _start_game.rpc()


# ── RPCs to clients ───────────────────────────────────────────────────────────

@rpc("authority", "reliable")
func _sync_lobby_state(state: Dictionary) -> void:
    # Client receives this once on join to populate its local lobby UI.
    pass  # Override in client-side lobby UI script


@rpc("authority", "reliable", "call_local")
func _notify_player_joined(peer_id: int, username: String) -> void:
    print("[Lobby] %s joined (id %d)" % [username, peer_id])


@rpc("authority", "reliable", "call_local")
func _notify_player_left(peer_id: int, username: String) -> void:
    print("[Lobby] %s left (id %d)" % [username, peer_id])


@rpc("authority", "reliable", "call_local")
func _notify_ready_changed(peer_id: int, is_ready: bool) -> void:
    print("[Lobby] Peer %d ready: %s" % [peer_id, str(is_ready)])


@rpc("authority", "reliable")
func _kick_peer(reason: String) -> void:
    push_error("[Lobby] Kicked: %s" % reason)
    multiplayer.multiplayer_peer.close()


@rpc("authority", "reliable", "call_local")
func _start_game() -> void:
    # Transition to gameplay scene on all peers.
    get_tree().change_scene_to_file("res://scenes/game.tscn")
```


---

