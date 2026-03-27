# Render-Architektur — Austauschbare GUI- und Browser-Engines

[← Zurück zum Index](../INDEX.md) | [Desktop](../programme/desktop/README.md)

---

## Warum eine Abstraktionsschicht?

FreeSynergy soll langfristig wartbar sein. GUI-Bibliotheken ändern sich, neue entstehen.
Eine direkte Abhängigkeit auf iced oder Bevy im App-Code würde bedeuten: bei einem Wechsel
muss alles neu geschrieben werden.

Stattdessen definiert `fs-render` einen Satz von **Traits** — die App kennt nur diese Traits.
Die Engine dahinter ist austauschbar. Zwei Engines gibt es von Anfang an:
`fs-gui-engine-iced` (klassisch) und `fs-gui-engine-bevy` (3D-fähig).
Das beweist, dass die API wirklich engine-unabhängig ist.

Dasselbe gilt für den Browser: `fs-web-engine` ist der Trait, `fs-web-engine-servo` ist die erste Implementierung.

---

## Implementierungs-Status

| Repo | Status |
|---|---|
| `fs-render` | ✅ G2.1 — alle Traits implementiert, 19 Tests grün |
| `fs-gui-engine-iced` | ✅ G2.2 — IcedEngine/IcedWindow/IcedWidget/IcedTheme, 16 Tests grün |
| `fs-gui-engine-bevy` | G2.3 — geplant |
| `fs-web-engine` | G2.5 — geplant |
| `fs-web-engine-servo` | G2.6 — geplant |

---

## GUI-Schichtmodell

```
┌─────────────────────────────────────────────────┐
│  fs-desktop / fs-apps                           │
│  (Shell, Sideboard, alle Apps)                  │
│  → importiert nur fs-render Traits              │
├─────────────────────────────────────────────────┤
│  fs-render                                      │
│  RenderEngine · FsWidget · FsWindow             │
│  FsTheme · FsEvent · AnimationSet               │
├──────────────────┬──────────────────────────────┤
│ fs-gui-engine-   │ fs-gui-engine-bevy           │
│ iced             │                              │
│ (iced 0.13-Basis)│ (Bevy ECS, 3D-fähig)        │
└──────────────────┴──────────────────────────────┘
```

---

## Browser-Schichtmodell

```
┌─────────────────────────────────────────────────┐
│  fs-browser  (App-Frontend)                     │
│  → importiert nur fs-web-engine Traits          │
├─────────────────────────────────────────────────┤
│  fs-web-engine                                  │
│  WebEngine · WebView · WebPlugin                │
├─────────────────────────────────────────────────┤
│  fs-web-engine-servo                            │
│  (Servo, SpiderMonkey JS, MPL-2.0)              │
└─────────────────────────────────────────────────┘
```

---

## Repos

| Repo | Lizenz | Rolle |
|---|---|---|
| `fs-render` | MIT | GUI-Abstraktions-Traits |
| `fs-gui-engine-iced` | MIT | iced-Engine; libcosmic-Ausbau geplant (Apache-2.0/MIT dual) |
| `fs-gui-engine-bevy` | MIT | Bevy-Engine (Apache-2.0/MIT dual) |
| `fs-web-engine` | MIT | Browser-Engine-Abstraktions-Traits |
| `fs-web-engine-servo` | MIT | Servo-Implementierung (Servo: MPL-2.0) |

---

## fs-render — GUI-Abstraktion

### Kern-Traits

```rust
/// Die Engine selbst — Lifecycle, Window-Erstellung, Event-Dispatch
pub trait RenderEngine: Send + Sync + 'static {
    type Window: FsWindow;
    type Widget: FsWidget;
    type Theme: FsTheme;

    fn name(&self) -> &str;
    fn version(&self) -> &str;
    fn create_window(&self, config: WindowConfig) -> Self::Window;
    fn apply_theme(&self, theme: &Self::Theme);
    fn dispatch_event(&self, event: FsEvent);
    fn shutdown(&self);
}

/// Jedes UI-Element — Engines wrappen ihre nativen Widgets in diesem Trait
pub trait FsWidget: Send + Sync {
    fn widget_id(&self) -> &str;
    fn is_enabled(&self) -> bool;
    fn set_enabled(&mut self, enabled: bool);
}

/// Jedes Fenster / App-Hauptfenster
pub trait FsWindow: Send + Sync {
    fn title(&self) -> &str;
    fn is_visible(&self) -> bool;
    fn show(&mut self);
    fn hide(&mut self);
    fn minimize(&mut self);
    fn restore(&mut self);
    fn close(&mut self);
    fn set_title(&mut self, title: String);
    fn on_event(&mut self, event: FsEvent);
}

/// Theme-Schnittstelle — engine-neutral
pub trait FsTheme: Send + Sync + Clone + Default {
    fn name(&self) -> &str;
    fn primary_color(&self) -> Color;
    fn background_color(&self) -> Color;
    fn text_color(&self) -> Color;
    fn accent_color(&self) -> Color;
    fn border_radius(&self) -> f32;
    fn font_size_base(&self) -> f32;
}
```

### Widget-Deskriptoren

`fs-render` definiert auch konkrete Deskriptoren die Engine-unabhängig beschreiben
welche Widgets existieren sollen:

```rust
pub struct ButtonWidget   { pub id, label, enabled, action }
pub struct TextInputWidget{ pub id, placeholder, value, enabled }
pub struct ListWidget     { pub id, items, selected_index }
```

### FsEvent

```rust
pub enum FsEvent {
    Key(KeyEvent),        // Taste + Modifier-Flags
    Mouse(MouseEvent),    // Click, Move, Scroll
    Window(WindowEvent),  // Resized, Focused, Minimized, CloseRequested, ...
    Custom(CustomEvent),  // Action, TextChanged, SelectionChanged
}
```

### AnimationSet

Animationen sind keine hartkodierten Werte — sie sind **Store-Pakete**:

```rust
pub struct AnimationSet {
    pub id: String,
    pub definitions: Vec<AnimationDefinition>,
}

pub struct AnimationDefinition {
    pub name: String,
    pub animation_type: AnimationType,  // SlideLeft, Fade, Scale, Custom(String), ...
    pub duration_ms: u32,
    pub easing: EasingFunction,
}
```

`AnimationType::Custom(name)` erlaubt WASM-basierte User-Animationen via `fs-plugin-sdk`.
AnimationSets werden wie Icon-Sets als Pakete im Store veröffentlicht und installiert.

### RenderCtx

```rust
pub struct RenderCtx {
    pub locale: String,           // z.B. "de", "en"
    pub animation: AnimationConfig,
}
```

---

## fs-gui-engine-iced ✅

Implementiert alle `fs-render`-Traits auf Basis von **iced 0.13**.

**Aktuelle Implementierung:**

| Struct | Trait |
|---|---|
| `IcedEngine` | `RenderEngine` |
| `IcedWindow` | `FsWindow` — Descriptor + Event-Queue |
| `IcedWidget` | `FsWidget` — ID + enabled-State |
| `IcedTheme` | `FsTheme` — wraps `iced::Theme`, FS Default = Dark + Cyan |

**IcedWindow — Event-Queue-Pattern:**

Da iced ein funktionales Elm-Modell hat, speichert `IcedWindow` pending Events
in einer Queue. Die iced-Application liest diese Queue im `update()`-Loop:

```rust
let events = window.drain_events(); // gibt Vec<FsEvent> zurück und leert Queue
```

**libcosmic-Ausbau (geplant für G2.8):**

Die vollständige libcosmic-Integration (Pop!_OS COSMIC Design System, System-Palette,
Portal-Integration) kommt wenn `fs-desktop` in Phase G2.8 auf diese Engine umgestellt wird.

**Standard-Engine** für fs-desktop. Gut für:
- Klassische UI (Formulare, Listen, Dialoge)
- Sideboard, Settings, Store-Browser
- Alle normalen Apps

---

## fs-gui-engine-bevy (geplant — G2.3)

Implementiert alle `fs-render`-Traits auf Basis von **Bevy** (ECS-Game-Engine).

**Bevy** ermöglicht:
- 3D-Visualisierungen im Desktop (Workspace-Ansichten, Daten-Graphen)
- Flüssige Animationen und Übergänge
- Shader-basierte Effekte
- Mischform: Bevy für 3D-Canvas, iced-Widgets für Dialoge (via Overlay)

**Einsatz:** Optionale Engine. Wer einen 3D-Desktop will, wählt Bevy.
Die Apps brauchen keine Änderung — nur der Engine-Feature-Flag ändert sich.

---

## fs-web-engine — Browser-Abstraktion (geplant — G2.5)

### Kern-Traits (geplant)

```rust
pub trait WebEngine {
    fn load(&self, url: &FsUrl) -> Result<()>;
    fn reload(&self) -> Result<()>;
    fn navigate_back(&self) -> Result<()>;
    fn navigate_forward(&self) -> Result<()>;
    fn execute_js(&self, script: &str) -> Result<JsValue>;
    fn install_plugin(&self, plugin: WasmPlugin) -> Result<()>;
}

pub trait WebView {
    fn embed(&self, window: &dyn FsWindow) -> Result<()>;
    fn set_zoom(&self, factor: f32);
}
```

### Plugin-System

Browser-Plugins sind **WASM-Module** via `fs-plugin-sdk` — kein WebExtensions-Nachbau.
Das passt ins bestehende Plugin-System und ist auf allen Plattformen portierbar.

---

## fs-web-engine-servo (geplant — G2.6)

**Servo** ist ein Browser-Engine in Rust (Mozilla-Projekt, jetzt eigenständig).

| Feature | Status |
|---|---|
| HTML/CSS Rendering | ✅ vollständig |
| JavaScript (SpiderMonkey) | ✅ vollständig |
| WebGL | ✅ |
| WebAssembly | ✅ |
| WebExtensions API | ❌ — eigenes WASM-Plugin-System stattdessen |

**Lizenz:** MPL-2.0 — modifizierte Servo-Dateien bleiben MPL. Solange wir Servo nur nutzen (keine internen Patches), kein Problem.

**Zukunft:** Wenn **Blitz** (Dioxus-Team, MIT) reif ist, kann `fs-web-engine-blitz` als Alternative entstehen. fs-browser braucht dafür keine Änderung.

---

## Engine-Auswahl

### Compile-Zeit (Feature-Flags)

```toml
# fs-desktop/Cargo.toml
[features]
default      = ["engine-iced"]
engine-iced  = ["dep:fs-gui-engine-iced"]
engine-bevy  = ["dep:fs-gui-engine-bevy"]
```

### Laufzeit (geplant)

Engine-Auswahl über Settings → aus dem installierten AnimationSet und Engine-Paket wird automatisch die richtige Engine geladen.

---

## Plattform-Unterstützung

| Platform | GUI-Engine | Status |
|---|---|---|
| Linux | iced | ✅ primär |
| macOS | iced | ✅ |
| Windows | iced | ✅ |
| Linux 3D | Bevy | Geplant |
| Android/iOS | TBD | Geplant |

---

Weiter: [Desktop](../programme/desktop/README.md) | [Browser](../programme/browser/README.md) | [Themes](../konzepte/themes.md)
