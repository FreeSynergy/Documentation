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
│ (libcosmic-Basis)│ (Bevy ECS, 3D-fähig)        │
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
| `fs-gui-engine-iced` | MIT | iced-Engine; libcosmic als Basis (Apache-2.0/MIT dual) |
| `fs-gui-engine-bevy` | MIT | Bevy-Engine (Apache-2.0/MIT dual) |
| `fs-web-engine` | MIT | Browser-Engine-Abstraktions-Traits |
| `fs-web-engine-servo` | MIT | Servo-Implementierung (Servo: MPL-2.0) |

---

## fs-render — GUI-Abstraktion

### Kern-Traits

```rust
/// Die Engine selbst — Lifecycle, Rendering-Loop
pub trait RenderEngine {
    fn run(&self, app: Box<dyn FsApp>) -> Result<()>;
    fn capabilities(&self) -> EngineCapabilities;
}

/// Jedes UI-Element
pub trait FsWidget {
    fn render(&self, ctx: &RenderContext) -> WidgetTree;
}

/// Jedes Fenster / App-Hauptfenster
pub trait FsWindow {
    fn title(&self) -> &str;
    fn on_minimize(&mut self);
    fn on_restore(&mut self);
    fn on_close(&mut self) -> CloseAction;
    fn layout(&self) -> WindowLayout;
}

/// Theme-Schnittstelle
pub trait FsTheme {
    fn colors(&self) -> &ColorPalette;
    fn fonts(&self) -> &FontSet;
    fn spacing(&self) -> &SpacingScale;
    fn animation_set(&self) -> &AnimationSet;
}
```

### AnimationSet

Animationen sind keine hartkodierten Werte — sie sind **Store-Pakete**:

```rust
pub struct AnimationSet {
    pub name: String,
    pub window_open:  Animation,
    pub window_close: Animation,
    pub tab_switch:   Animation,
    pub app_launch:   Animation,
    // ...
}

pub struct Animation {
    pub kind: AnimationKind,   // SlideLeft, SlideRight, Fade, Scale, Custom(WasmModule)
    pub duration_ms: u32,
    pub easing: EasingFn,
}
```

`Custom(WasmModule)` erlaubt WASM-basierte User-Animationen via `fs-plugin-sdk`.
AnimationSets werden wie Icon-Sets als Pakete im Store veröffentlicht und installiert.

---

## fs-gui-engine-iced

Implementiert alle `fs-render`-Traits auf Basis von **iced** + **libcosmic** (System76).

**libcosmic** liefert:
- Fertige Widgets (Buttons, Listen, Dialoge, Tabs, Sideboard-Elemente)
- Theming-System (CSS-Variablen-ähnlich)
- Fensterverhalten (Titelleiste, Resize, Snap)
- Dark/Light-Mode-Unterstützung

**Lizenz:** libcosmic ist Apache-2.0/MIT dual — problemlos nutzbar.

**Standard-Engine** für fs-desktop. Gut für:
- Klassische UI (Formulare, Listen, Dialoge)
- Sideboard, Settings, Store-Browser
- Alle normalen Apps

---

## fs-gui-engine-bevy

Implementiert alle `fs-render`-Traits auf Basis von **Bevy** (ECS-Game-Engine).

**Bevy** ermöglicht:
- 3D-Visualisierungen im Desktop (Workspace-Ansichten, Daten-Graphen)
- Flüssige Animationen und Übergänge
- Shader-basierte Effekte
- Mischform: Bevy für 3D-Canvas, iced-Widgets für Dialoge (via Overlay)

**Einsatz:** Optionale Engine. Wer einen 3D-Desktop will, wählt Bevy.
Die Apps brauchen keine Änderung — nur der Engine-Feature-Flag ändert sich.

---

## fs-web-engine — Browser-Abstraktion

### Kern-Traits

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

## fs-web-engine-servo

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
default  = ["engine-iced"]
engine-iced  = ["dep:fs-gui-engine-iced"]
engine-bevy  = ["dep:fs-gui-engine-bevy"]
```

### Laufzeit (geplant)

Engine-Auswahl über Settings → aus dem installierten AnimationSet und Engine-Paket wird automatisch die richtige Engine geladen.

---

## Plattform-Unterstützung

| Platform | GUI-Engine | Status |
|---|---|---|
| Linux | iced (libcosmic) | ✅ primär |
| macOS | iced | ✅ |
| Windows | iced | ✅ |
| Linux 3D | Bevy | Geplant |
| Android/iOS | TBD | Geplant |

---

Weiter: [Desktop](../programme/desktop/README.md) | [Browser](../programme/browser/README.md) | [Themes](../konzepte/themes.md)
