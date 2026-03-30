# fs-web-engine

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-web-engine` ist die Browser-Engine-Abstraktion für FreeSynergy — eine reine Library (kein Daemon).
Sie definiert Traits für Browser-Engines und Web-Views.
Konkrete Implementierungen leben in Adapter-Repos (`fs-web-engine-servo`).

---

## Architektur

```
WebEngine-Trait    ← Abstract Factory Pattern
WebView-Trait      ← Navigation, JS-Ausführung
WebPlugin-Trait    ← WASM-Plugins, Request-Interception
WebCapabilities    ← Feature-Flags der Engine
```

---

## Traits

| Trait            | Beschreibung                                              |
|------------------|-----------------------------------------------------------|
| `WebEngine`      | `create_view`, `capabilities`, `version`                  |
| `WebView`        | `navigate`, `reload`, `go_back`, `execute_js`, `url`      |
| `WebPlugin`      | `init`, `handle_request`, `name`, `metadata`              |
| `WebCapabilities`| `javascript`, `wasm`, `webgl`, `media`, `storage`         |

---

## Repo

- Lokal: `/home/kal/Server/fs-web-engine/`
- GitHub: `git@github.com:FreeSynergy/fs-web-engine.git`

---

Weiter: [fs-web-engine-servo](fs-web-engine-servo.md) | [← Index](../INDEX.md)
