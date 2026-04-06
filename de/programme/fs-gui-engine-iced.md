# fs-gui-engine-iced

**iced / libcosmic Render-Engine für FreeSynergy**

Implementiert alle `fs-render`-Traits auf Basis der
[iced](https://iced.rs/) 0.13 GUI-Bibliothek.  Consumer-Code importiert **nur**
`fs-render` — niemals dieses Crate direkt.

---

## Architektur

```
fs-render (Traits — keine Engine-Abhängigkeit)
    └── fs-gui-engine-iced
            engine.rs    — IcedEngine  → implementiert RenderEngine
            window.rs    — IcedWindow  → implementiert FsWindow
            widget.rs    — IcedWidget  → implementiert FsWidget
            theme.rs     — IcedTheme   → implementiert FsTheme
            layout.rs    — IcedLayoutInterpreter: LayoutDescriptor → iced Elements
            mvu.rs       — Elm-Muster (Model–View–Update), MvuApp
            navigation.rs— CornerMenu / SideMenu Renderer (G1.2)
            capability.rs— "render.engine.iced" Capability für fs-registry
            keys.rs      — FTL-Konstanten (NAV_CORNER_INDICATOR, …)
```

**Adapter Pattern**: `IcedEngine` adaptiert die iced-API an die
`RenderEngine`-Schnittstelle.  Der Desktop-Shell kennt nur `RenderEngine` —
die konkrete Engine wird über Cargo-Feature-Flags gewählt.

---

## Module

### `IcedEngine` — `RenderEngine`-Implementierung

Einstiegspunkt für alle FreeSynergy-Applikationen, die das iced-Backend nutzen.

| Methode | Beschreibung |
|---|---|
| `new()` | Erstellt Engine mit FreeSynergy-Default-Theme |
| `create_window(cfg)` | Erstellt einen `IcedWindow`-Deskriptor |
| `apply_theme(theme)` | Speichert das aktive Theme für den nächsten Render-Tick |
| `set_context(ctx)` | Aktualisiert Locale, Theme-Name, Feature-Flags |
| `app_context()` | Liest den aktuellen `AppContext`-Snapshot |
| `dispatch_event(evt)` | Leitet ein `FsEvent` in die iced-Event-Schleife |
| `run()` | Lifecycle-Hook (no-op; Start via `run_app`) |
| `shutdown()` | Sendet Abbruch-Signal an iced |
| `run_app(title, update, view)` | Startet die typisierte iced-Event-Schleife |

### `IcedWindow` — `FsWindow`-Implementierung

Hält den Fensterzustand zwischen Render-Ticks.  Mutationen werden als
`FsEvent`-Liste gepuffert und von der iced-`update`-Funktion geleert.

```rust
let mut win = IcedWindow::new("Meine App");
win.minimize();
let events = win.drain_events(); // enthält FsEvent::Window(Minimized)
```

### `IcedWidget` — `FsWidget`-Implementierung

Widget-Deskriptor: ID + enabled-Flag.  Die iced-View-Funktion liest den
Deskriptor und baut das konkrete `iced::Element`.

### `IcedTheme` — `FsTheme`-Implementierung

Wrapper um `iced::Theme`.  Alle `FsTheme`-Abfragen leiten ihre Werte aus
der iced-Palette ab — beide bleiben so automatisch synchron.

```rust
let theme = IcedTheme::fs_default();   // FreeSynergy Default (Dark, Cyan)
let bg = theme.background_color();     // Color::rgb(0.08, 0.08, 0.12)
```

---

## MVU-Muster (`mvu`-Modul)

Das Elm-Pattern trennt drei Belange sauber:

| Schicht | Typ | Beschreibung |
|---|---|---|
| **Model** | `S: Default` | Unveränderlicher Applikationszustand |
| **Update** | `fn(&mut S, M) -> Task<M>` | Reine Zustandsübergangs-Funktion |
| **View** | `fn(&S) -> Element<M>` | Reine Render-Funktion |

```rust
use fs_gui_engine_iced::mvu::MvuApp;

MvuApp::new("Titel", update_fn, view_fn).run().unwrap();
```

`MvuApp::run` delegiert intern an `iced::application(…).run()`.
Consumer-Crates brauchen kein `iced` in ihrem `Cargo.toml`.

---

## Navigation-Renderer (G1.2)

Modul `navigation.rs` — Interpreter Pattern: liest `CornerMenuDescriptor` /
`SideMenuDescriptor` (aus `fs-render`) und produziert iced-Element-Bäume.
Kein iced-Import entweicht in Consumer-Code.

### `MenuConfig`

Laufzeit-Konfiguration für beide Menü-Typen.

| Feld | Default | Beschreibung |
|---|---|---|
| `icon_size` | `32.0` | Basis-Item-Höhe (px) |
| `max_icon_size` | `48.0` | Max-Höhe bei Hover (Magnification) |
| `spread` | `2.0` | Falloff-Breite der Magnification |
| `indicator_radius` | `20.0` | Indikator-Radius (px) |
| `accent` | `#00E5E5` | FreeSynergy-Cyan für Indikator + aktive Items |

### Zustands-Typen (MVU-kompatibel)

```rust
pub struct CornerMenuState { pub open: bool, pub hovered_idx: Option<usize> }
pub struct SideMenuState   { pub open: bool, pub hovered_idx: Option<usize> }
```

### `NavMessage`

```
CornerMenuToggle(Corner)               — Menü öffnen/schließen
CornerMenuItemEntered(Corner, usize)   — Hover: Magnification-Index setzen
CornerMenuItemLeft(Corner)             — Hover verlassen
CornerMenuAction(Corner, String)       — Item aktiviert (Action-String)

SideMenuToggle(Side)
SideMenuItemEntered(Side, usize)
SideMenuItemLeft(Side)
SideMenuAction(Side, String)
```

### State-Updater

```rust
// Fan-out: jeder Updater ignoriert Nachrichten für andere Menüs
update_corner_menu(&mut state, corner, &msg);
update_side_menu  (&mut state, side,   &msg);
```

### Render-Funktionen

```
render_corner_menu(descriptor, state, config) → Element<NavMessage>
render_side_menu  (descriptor, state, config) → Element<NavMessage>
```

**Visuelles Verhalten:**
- Indikatoren: asymmetrische `border-radius` → Viertelkreis (Corner) / Halbkreis (Side)
- Items: Column transparenter Buttons; Höhe skaliert per Hover-Magnification (exponentieller Falloff)
- Scroll-Fallback: `scrollable` ab `SCROLL_THRESHOLD = 8` Items
- Sub-Items: `▶`-Suffix auf Label (Basis für weitere Ebenen in G1.5)
- Icon-Hintergründe: immer transparent

---

## Capability-Registrierung

Beim Start registriert die Engine die Capability **`render.engine.iced`** in
`fs-registry`, damit Shell und App-Launcher wissen, welche Render-Engines
verfügbar sind.

```rust
use fs_gui_engine_iced::{IcedCapability, CAPABILITY_ID};

let cap = IcedCapability::descriptor();
assert_eq!(cap.id, "render.engine.iced");
```

---

## i18n

Alle Strings sind als FTL-Keys in `fs-i18n` hinterlegt:

- `fs-i18n/locales/en/gui-engine-iced.ftl`
- `fs-i18n/locales/de/gui-engine-iced.ftl`

Schlüssel-Präfix: `gui-iced-`

---

## Optional Features

Feature-Flags aktivieren Desktop-Integration ohne Pflichtabhängigkeit.

| Feature | Aktiviert | Crate | Zweck |
|---|---|---|---|
| `wayland` | `iced_layershell 0.17` | Wayland Layer Shell | Panel, Dock, Overlays (Corner/Side menus als echte Layer-Windows) |
| `portals` | `ashpd 0.13` | XDG Portals (D-Bus) | File Picker, Notifications, Screenshots, OpenURI — desktop-agnostisch |
| `theme-ext` | `palette 0.7` | Farbmathe | Oklch-Interpolation für Dark/Light-Übergänge, `FsTheme`-Erweiterung |
| `icon-lookup` | `freedesktop-icons 0.4` | Icon-Theme-Lookup | Löst Icon-Namen per FreeDesktop-Standard zu Pfaden auf |
| `desktop` | alle vier | — | Convenience: aktiviert alle Desktop-Integration-Features |

Kein COSMIC, kein libcosmic. Alle Crates sind standalone.

---

## Qualitäts-Gates

```
cargo clippy --all-targets -- -D warnings   → 0 Fehler
cargo fmt --check                           → sauber
cargo test                                  → 48+ Tests, alle grün
cargo build --release                       → fehlerfrei
```

---

## Abhängigkeiten

| Crate | Version | Rolle |
|---|---|---|
| `fs-render` | path | Trait-Definitionen (RenderEngine, FsWidget, …) |
| `iced` | 0.14 | GUI-Framework — reaktives Rendering, native Animation API, IME |
| `iced_aw` | 0.13 | Zusatz-Widgets: Badge, Card, Spinner, TabBar, ContextMenu |
| `iced_layershell` | 0.17 | Wayland Layer Shell *(optional: `wayland`)* |
| `ashpd` | 0.13 | XDG Portals *(optional: `portals`)* |
| `palette` | 0.7 | Farbmathe *(optional: `theme-ext`)* |
| `freedesktop-icons` | 0.4 | Icon-Lookup *(optional: `icon-lookup`)* |

---

Siehe auch: `fs-render` (Trait-Definitionen) · `fs-gui-engine-bevy` (Bevy-Implementierung)
