# Manager

[← Zurück zum Index](../INDEX.md) | [Store](../programme/store/README.md) | [Inventory](inventory.md)

---

## Was ein Manager ist

Ein **Manager** ist der Kleber zwischen dem Store (was verfügbar ist) und den Programmen (was sie brauchen). Anstatt dass jedes Programm selbst entscheidet wie es Sprachen, Themes oder Apps verwaltet, ruft es den zuständigen Manager auf.

```
Store ←→ Manager ←→ Programme / UI
```

- Der Manager liest den aktuellen Zustand aus dem Store
- Der Manager bietet eine fertige UI-Komponente an (z.B. Sprachauswahl)
- Der Manager schreibt Änderungen zurück in den Store (wenn berechtigt)
- Settings ruft Manager auf — hat selbst keine Logik für diese Bereiche
- Das Inventory empfängt die fertigen, installierten Zustände

## Warum Manager?

Ohne Manager verwaltet jedes Programm seinen eigenen Zustand:
- Desktop hat eine eigene Sprachauswahl
- Node hat eine eigene Sprachauswahl
- Beide können auseinanderlaufen

Mit Manager:
- Einmal geändert → überall aktualisiert
- Kein Dropdown im UI — das Element gehört dem LanguageManager
- Konsistenz durch Ownership

## Kontext-Bewusstsein

Nicht jeder Manager macht überall Sinn. Bilder in einem Terminal ergeben keinen Sinn — Sprache schon. Jedes Programm bindet nur die Manager ein, die es wirklich braucht.

## Verfügbare Manager

Alle Manager leben im Repo **FreeSynergy.Managers** (`git@github.com:FreeSynergy/Managers.git`).

| Unterordner | Crate | Zuständigkeit |
|---|---|---|
| `language/` | `fsn-manager-language` | Aktive Sprache lesen, setzen, UI-Picker |
| `theme/` | `fsn-manager-theme` | Aktives Theme lesen, setzen, UI-Picker |
| `container_app/` | `fsn-manager-container-app` | Container-Apps installieren, starten, stoppen, entfernen → [Container App Manager](../programme/container_app/README.md) |
| `icons/` | `fsn-manager-icons` | Icon-Sets kennen, Pfade auflösen, UI Icon-Picker |
| `bots/` | `fsn-manager-bot` | Bots benutzen: Broadcasts senden, Subscriptions, Gatekeeper → [BotManager](../programme/botmanager/README.md) |

## Verwendung

```rust
use fsn_manager_language::LanguageManager;

let mgr = LanguageManager::new();
let lang = mgr.active();       // aktuelle Sprache
let all  = mgr.available();    // alle Sprachen
mgr.set_active("de")?;         // Sprache wechseln → schreibt in den Store
```

Der Manager liefert immer ein konsistentes Ergebnis — unabhängig davon wo er aufgerufen wird.

## Berechtigungen

Manager können nur dann in den Store schreiben, wenn sie die entsprechende Berechtigung haben. Die Rechte-Kaskade gilt wie überall: Schreib-Rechte können nur vom Node-Owner vergeben werden.

Siehe: [Rechte-Kaskade](rechte.md)

## Icon-Manager und Icon-Sets

Der Icon-Manager liest das `manifest.toml` aus **FreeSynergy.Icons** und weiß welche Sets verfügbar sind. Er löst Icon-Namen zu Dateipfaden auf und stellt einen Icon-Picker zur Verfügung.

Neue Icon-Sets werden in FreeSynergy.Icons als eigener Ordner hinzugefügt (z.B. `homarrlabs/`, `simpleicons/`). Der Icon-Manager erkennt sie automatisch über das Manifest.

Siehe: [FreeSynergy.Icons](https://github.com/FreeSynergy/Icons)

---

Weiter: [Store](../programme/store/README.md) | [Rechte-Kaskade](rechte.md) | [Themes](themes.md) | [BotManager](../programme/botmanager/README.md) | [Container App Manager](../programme/container_app/README.md)
