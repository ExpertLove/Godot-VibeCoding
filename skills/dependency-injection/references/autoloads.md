# Autoloads as Singletons

Reference for `skills/dependency-injection/SKILL.md` — full Autoload pattern: AudioManager example with one-shot SFX and crossfade music. GDScript.

> ← Back to [SKILL.md](../SKILL.md)

---
## 3. Autoloads as Singletons

Autoloads are appropriate for services that are **genuinely global** and have **no meaningful test double** (e.g., audio playback, OS settings, platform APIs). Avoid using them as a dumping ground for any shared state.

**When appropriate:**
- Audio mixer / SFX player
- User settings / ConfigFile wrapper
- Platform-specific services (achievements, IAP)
- Scene transition manager

**Dangers:**
- Hidden dependencies — callers do not declare what they need
- Autoload ordering bugs — A depends on B, but B isn't ready yet
- Test difficulty — autoloads must be fully operational during unit tests

### GDScript (`autoloads/audio_manager.gd`)

```gdscript
extends Node

## Plays a one-shot sound effect by key.
func play_sfx(key: String) -> void:
    var stream: AudioStream = _sfx_library.get(key)
    if stream == null:
        push_warning("AudioManager: unknown sfx key '%s'" % key)
        return
    var player := AudioStreamPlayer.new()
    player.stream = stream
    player.finished.connect(player.queue_free)
    add_child(player)
    player.play()


## Plays background music, crossfading from the current track.
func play_music(key: String, crossfade_time: float = 0.5) -> void:
    pass  # implementation omitted for brevity


var _sfx_library: Dictionary = {}


func _ready() -> void:
    _load_sfx_library()


func _load_sfx_library() -> void:
    # Populate _sfx_library from a Resource or folder scan
    pass
```


---

