# Server Configuration

Reference for `skills/dedicated-server/SKILL.md` — command-line argument parsing for headless server (port, max-players, tick-rate, log-level), GDScript.

> ← Back to [SKILL.md](../SKILL.md)

---
## 5. Server Configuration

### Command Line Argument Parsing

### GDScript

```gdscript
# server_config.gd — autoload named ServerConfig, parsed before any other autoload logic
extends Node

var port: int        = 7777
var max_players: int = 8
var tick_rate: int   = 60
var log_level: int   = 1   # 0 = quiet, 1 = info, 2 = verbose


func _ready() -> void:
    _parse_args()
    _load_config_file("user://server.cfg")
    _apply_env_vars()

    if OS.has_feature("dedicated_server") or DisplayServer.get_name() == "headless":
        print("[Config] port=%d  max_players=%d  tick_rate=%d" % [port, max_players, tick_rate])


func _parse_args() -> void:
    var args := OS.get_cmdline_args()
    var i := 0
    while i < args.size():
        match args[i]:
            "--port":
                if i + 1 < args.size():
                    port = int(args[i + 1])
                    i += 1
            "--max-players":
                if i + 1 < args.size():
                    max_players = int(args[i + 1])
                    i += 1
            "--tick-rate":
                if i + 1 < args.size():
                    tick_rate = int(args[i + 1])
                    i += 1
            "--log-level":
                if i + 1 < args.size():
                    log_level = int(args[i + 1])
                    i += 1
        i += 1


func _load_config_file(path: String) -> void:
    var cfg := ConfigFile.new()
    var err := cfg.load(path)
    if err != OK:
        return  # No config file — defaults remain in effect

    port        = cfg.get_value("server", "port",        port)
    max_players = cfg.get_value("server", "max_players", max_players)
    tick_rate   = cfg.get_value("server", "tick_rate",   tick_rate)
    log_level   = cfg.get_value("server", "log_level",   log_level)
    print("[Config] Loaded config from: %s" % path)


func _apply_env_vars() -> void:
    # Environment variables override the config file but are overridden by CLI args.
    var env_port := OS.get_environment("SERVER_PORT")
    if env_port != "":
        port = int(env_port)

    var env_max := OS.get_environment("SERVER_MAX_PLAYERS")
    if env_max != "":
        max_players = int(env_max)

    var env_tick := OS.get_environment("SERVER_TICK_RATE")
    if env_tick != "":
        tick_rate = int(env_tick)
```

**Example config file (`server.cfg`):**

```ini
[server]
port=7777
max_players=16
tick_rate=60
log_level=1
```

**Example launch:**

```bash
./my_game_server.x86_64 --headless --port 7778 --max-players 4 --tick-rate 30
```


---

