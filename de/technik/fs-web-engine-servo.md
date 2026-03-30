# fs-web-engine-servo

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-web-engine-servo` implementiert die `fs-web-engine`-Traits via [Servo](https://servo.org/)
(SpiderMonkey JS, MPL-2.0 Lizenz).
Registriert die Capability `web.engine.servo` in `fs-registry`.

> **Lizenz**: Servo ist MPL-2.0 — für die Kombinierbarkeit in FreeSynergy beachten.

---

## Architektur

```
ServoWebEngine     ← implementiert WebEngine-Trait; Adapter Pattern
ServoWebView       ← implementiert WebView-Trait (Navigation, JS)
ServoPluginRegistry ← Composite Pattern für WASM-Plugins
```

**Feature-Flag**: `servo` aktiviert die echte Servo-Integration.
Ohne `servo`: `StubWebEngine` — kein Browser-Fenster, nützlich für Tests.

---

## Capability

| Feld           | Wert                        |
|----------------|-----------------------------|
| ID             | `web.engine.servo`          |
| Registriert in | `fs-registry`               |

---

## Repo

- Lokal: `/home/kal/Server/fs-web-engine-servo/`
- GitHub: `git@github.com:FreeSynergy/fs-web-engine-servo.git`

---

Weiter: [← Index](../INDEX.md)
