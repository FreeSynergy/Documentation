# Browser — Der eingebettete Web-Browser

[← Zurück zum Index](../../INDEX.md) | [Desktop](../desktop/README.md) | [Render-Architektur](../../architektur/render-architektur.md)

---

## Was der Browser macht

Der Browser ist eine eigenständige App die Web-Seiten anzeigt — als Standalone-Fenster oder eingebettet in den Desktop-Shell.

## Warum ein eigener Browser?

1. **Service-UIs anzeigen** — Kanidm, Forgejo, Outline haben Web-Interfaces. Statt den System-Browser zu öffnen, zeigt der FreeSynergy-Browser sie direkt im Desktop an.
2. **Integration** — Links aus Lenses, Search, Widgets öffnen im Browser statt extern.
3. **Downloads** — Dateien aus Service-UIs herunterladen, direkt in den S3-Storage.
4. **Einheitliches Erscheinungsbild** — Web-UIs sehen aus wie native Apps (Theme-Integration).

## Technisch

Der Browser nutzt die [Render-Abstraktionsschicht](../../architektur/render-architektur.md):

```
fs-browser (App)
    ↓ (nur Traits)
fs-web-engine (Abstraktions-Trait)
    ↓
fs-web-engine-servo (Servo + SpiderMonkey JS)
```

Der App-Code importiert **niemals** Servo direkt — nur `fs-web-engine`-Traits.
Engine-Wechsel (z.B. zu Blitz wenn reif) erfordern keine Änderung am Browser-Code.

### Browser-Engine: Servo

| Feature | Status |
|---|---|
| HTML/CSS | ✅ |
| JavaScript (SpiderMonkey) | ✅ |
| WebGL | ✅ |
| WebAssembly | ✅ |
| Plugins | ✅ WASM-basiert (fs-plugin-sdk) |

### Plugins

Browser-Plugins sind **WASM-Module** via `fs-plugin-sdk`.
Kein WebExtensions-Nachbau — das eigene Plugin-System ist cross-platform und sicherer.
Plugin-Pakete werden über den Store installiert.

## Module

| Modul | Zweck |
|---|---|
| `app.rs` | Root-Komponente (`BrowserApp`), URL-Leiste, Tabs |
| `model.rs` | `BrowserTab`, `BrowserConfig` — Domain-Modell |
| `bookmarks.rs` | `BookmarkManager` — CRUD, Serialisierung |
| `history.rs` | `HistoryManager` — chronologischer Verlauf |
| `search_engine.rs` | `SearchEngineRegistry` — URL-Builder je Engine |
| `plugin.rs` | `PluginRegistry` — WASM-Plugins laden + verwalten |

## Interfaces

| Interface | Funktion |
|---|---|
| URL-Leiste | Direkte URL-Eingabe |
| Tabs | Mehrere Seiten gleichzeitig |
| Service-Links | Klick auf Service im Container Manager → öffnet dessen Web-UI |
| Lens-Links | Klick auf Ergebnis in Lens → öffnet im Browser |
| Downloads | Direkt in S3-Storage |

## Repo

`git@github.com:FreeSynergy/fs-browser.git`

---

Weiter: [Render-Architektur](../../architektur/render-architektur.md) | [Desktop](../desktop/README.md) | [Lenses](../lenses/README.md)
