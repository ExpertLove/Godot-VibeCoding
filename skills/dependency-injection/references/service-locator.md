# Service Locator Pattern

Reference for `skills/dependency-injection/SKILL.md` — central registry autoload for runtime service lookup. GDScript implementation with typed helpers.

> ← Back to [SKILL.md](../SKILL.md)

---
## 5. Service Locator Pattern

A Service Locator is an autoload that acts as a runtime registry. Systems register themselves by name; consumers retrieve them by name. Unlike direct autoload access, the locator is the *only* hard dependency — everything else is swappable.

Useful for:
- Plugin architectures where systems opt in at runtime
- Swapping real implementations for stubs in tests
- Optional systems (e.g., analytics) that may or may not be present

### GDScript (`autoloads/service_locator.gd`)

```gdscript
extends Node

var _services: Dictionary = {}


## Registers a service under [param service_name].
## Call from the service's own _ready().
func register(service_name: String, instance: Object) -> void:
    if _services.has(service_name):
        push_warning("ServiceLocator: overwriting existing service '%s'" % service_name)
    _services[service_name] = instance


## Removes a service registration. Call from _exit_tree() of the service.
func unregister(service_name: String) -> void:
    _services.erase(service_name)


## Returns the service registered under [param service_name], or null.
## Cast the result to the expected type at the call site.
func get_service(service_name: String) -> Object:
    if not _services.has(service_name):
        push_warning("ServiceLocator: no service registered for '%s'" % service_name)
        return null
    return _services[service_name]


## Typed convenience helper — returns null and warns if the cast fails.
func get_typed(service_name: String, expected_type: Variant) -> Variant:
    var svc: Object = get_service(service_name)
    if svc == null:
        return null
    if not is_instance_of(svc, expected_type):
        push_warning(
            "ServiceLocator: '%s' is not an instance of %s" % [service_name, expected_type]
        )
        return null
    return svc
```

```gdscript
# audio_service.gd — self-registers on ready
extends Node
class_name AudioService

func _ready() -> void:
    ServiceLocator.register("audio", self)

func _exit_tree() -> void:
    ServiceLocator.unregister("audio")

func play_sfx(key: String) -> void:
    pass  # implementation


# consumer.gd — retrieves the service at runtime
func _ready() -> void:
    var audio := ServiceLocator.get_typed("audio", AudioService) as AudioService
    if audio != null:
        audio.play_sfx("pickup")
```




---

