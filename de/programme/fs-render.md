# fs-render

**GUI-Abstraktions-Traits für FreeSynergy**

Reine Trait-Library ohne jede Engine-Abhängigkeit.  Definiert die gemeinsame
Schnittstelle, gegen die alle GUI-Consumer-Crates programmieren.  Konkrete
Implementierungen leben in separaten Engine-Repos (`fs-gui-engine-iced`,
`fs-gui-engine-bevy`, `fs-web-engine-servo`).

---

## Architektur

```
fs-render  (Traits — kein iced / bevy / Dioxus)
    ├── engine.rs        — RenderEngine: Lebenszyklus + Widget/Window-Factory
    ├── widget.rs        — FsWidget: Widget-Abstraktion
    ├── window.rs        — FsWindow: Fenster-Lebenszyklus
    ├── theme.rs         — FsTheme: Theme-Schnittstelle
    ├── event.rs         — FsEvent: Eingabe- und Fenster-Events
    ├── view.rs          — FsView: Domain→Widget-Projektion (Adapter Pattern)
    ├── ctx.rs           — AppContext, RenderCtx, FeatureFlags
    ├── action.rs        — AppAction: Animation-Trigger-Enum (27 Varianten)
    ├── animation.rs     — AnimationSet, AnimationDefinition, AnimationRegistry
    ├── registry.rs      — AnimationActionMap: AppAction → AnimationDefinition
    ├── manager.rs       — ManagerLayout: Standard-Sidebar-Layout für Manager-Fenster
    ├── component.rs     — ComponentTrait, LayoutInterpreter, Standard-Komponenten
    ├── navigation.rs    — Descriptor-Traits: CornerMenu, SideMenu, HoverMagnification,
    │                      CompositeIcon, ProgramView, ProgramViewProvider  (G1.1)
    ├── activity.rs      — ActivityAction, ActivityCategory, ActivityEngine  (G1.6)
    ├── help_source.rs   — HelpSource, FocusObserver, FocusedElement  (G1.7)
    ├── ux_extras.rs     — BadgedIcon, WindowLayoutMode, QuickSwitchCommand,
    │                      NotificationCenter, WorkspaceProfileManager,
    │                      ClipboardHistory, WindowSnapManager, …  (G1.9)
    ├── render3d.rs      — Fs3dExtension: optionale 3-D-Erweiterung
    └── hot_reload.rs    — HotReloadWatcher: inotify-basierte Layout-Aktualisierung
```

**Kein Dioxus.** Alle Pfade der Rendering-Pipeline führen über diese Traits.
Consumer-Code importiert ausschließlich `fs-render` — niemals eine konkrete
Engine.

---

## Traits

### `RenderEngine`

Lifecycle-Trait für GUI-Engines.  Assoziierte Typen `Window`, `Widget` und
`Theme` sind die konkreten Engine-Typen, die alle fs-render-Traits erfüllen.

| Methode | Beschreibung |
|---|---|
| `name()` / `version()` | Engine-Identifikation |
| `create_window(cfg)` | Erstellt ein Fenster aus `WindowConfig` |
| `apply_theme(theme)` | Setzt das aktive Theme |
| `set_context(ctx)` | Übergibt `AppContext` (Locale, Theme-Name, Flags) |
| `dispatch_event(evt)` | Leitet ein `FsEvent` in die Event-Schleife |
| `run()` | Blockiert bis alle Fenster geschlossen sind |
| `shutdown()` | Graceful Shutdown |

`WindowConfig` Default: Titel `"FreeSynergy"`, 1280 × 800 px, resizable,
dekoriert.

---

### `FsWidget`

Widget-Deskriptor-Trait.  Implementiert von `ButtonWidget`, `TextInputWidget`
und `ListWidget` (Stub-Implementierungen im Crate — für Tests und als
Referenz).

| Methode | Beschreibung |
|---|---|
| `widget_id()` | Eindeutige Widget-ID |
| `is_enabled()` / `set_enabled(b)` | Aktivierungs-Zustand |
| `render_hint()` | Visueller Stil-Tipp für die Engine |
| `layout_hint()` | Layout-Präferenz (Fill, Shrink, …) |
| `style_hint()` | Zusätzlicher Style-Key |

---

### `FsWindow`

Fenster-Lebenszyklus-Trait: `title`, `size`, `show`, `hide`, `minimize`,
`restore`, `close`, `set_title`, `on_event(FsEvent)`.

---

### `FsTheme`

Theme-Schnittstelle: `name`, `primary_color`, `background_color`,
`text_color`, `border_radius`, `font_size_base`.

`DefaultTheme` liefert das FreeSynergy-Branding (Cyan `#00DFDF`,
dunkelgrauer Hintergrund, weiße Texte).

---

### `FsEvent`

```
FsEvent
    Key(KeyEvent)      — key + ctrl/alt/shift + pressed
    Mouse(MouseEvent)  — Click / Scroll / Move
    Window(WindowEvent)— Resized / Focused / Blurred / CloseRequested / Closed
    Custom(CustomEvent)— Action / TextChanged / SelectionChanged
```

---

## FsView-Pattern

```rust
// In my-domain/src/user/view.rs
use fs_render::view::FsView;
use fs_render::widget::{ButtonWidget, FsWidget};
use super::User;

impl FsView for User {
    fn view(&self) -> Box<dyn FsWidget> {
        Box::new(ButtonWidget {
            id:      format!("user-{}", self.id),
            label:   self.display_name.clone(),
            enabled: true,
            action:  "select".into(),
        })
    }
}
```

**Warum hier?** `FsWidget` ist der Rückgabetyp — Trait und Rückgabetyp müssen
im selben Crate liegen, sonst greift die Orphan-Rule.  Domain-Objekte selbst
importieren `fs-render` **nicht** direkt — nur ihre `view.rs`-Shims dürfen das.

---

## AppContext & RenderCtx

| Typ | Felder | Verwendung |
|---|---|---|
| `AppContext` | `locale`, `theme_name`, `feature_flags` | Wird über `RenderEngine::set_context` übergeben |
| `RenderCtx` | `locale`, `animation: AnimationConfig` | Leichtgewichtiger Kontext für Views/Widgets |
| `FeatureFlags` | `HashMap<String, bool>` | Runtime-Toggles für optionale Capabilities |

```rust
let mut ctx = AppContext::new("de", "FreeSynergy Default");
ctx.feature_flags.set("experimental.3d", true);
engine.set_context(ctx);
```

---

## Animations-System

### `AnimationSet`

Store-verteilbares Paket mit benannten `AnimationDefinition`-Einträgen.
Jede Definition enthält `AnimationType` (FadeIn, SlideIn, Scale, Bounce,
Shake, Pulse, Custom) und eine `EasingFunction`.

### `AnimationRegistry` (Registry Pattern)

Verwaltet geladene `AnimationSet`-Instanzen und eine `AnimationActionMap`
(Mapping `AppAction → AnimationDefinition`).

```rust
let reg = AnimationRegistry::new();          // lädt DefaultAnimationSet
let anim = reg.resolve(&AppAction::AppOpen); // → Some(&AnimationDefinition)
```

`DefaultAnimationSet` enthält 7 Standard-Animationen und deckt alle
`AppAction`-Varianten ab.

---

## 3-D-Erweiterung (`Fs3dExtension`)

Optionaler Trait, den Domain-Objekte zusätzlich zu `FsView` implementieren
können, wenn sie eine 3-D-Darstellung benötigen (z. B. für
`fs-gui-engine-bevy`).

```rust
pub trait Fs3dExtension {
    fn render_3d(&self) -> Fs3dDescriptor;
}
```

`Fs3dDescriptor` enthält `mesh_id`, eine 4×4-Transformationsmatrix und
`visible`.  Engines ohne 3-D-Support ignorieren den Trait vollständig — die
2-D-`FsView`-Pipeline ist davon unberührt.

---

## Qualitäts-Gates

```
cargo clippy --all-targets -- -D warnings   → 0 Fehler
cargo fmt --check                           → sauber
cargo test                                  → 40 Tests, alle grün
cargo build --release                       → fehlerfrei
```

---

## Abhängigkeiten

Keine Engine-Abhängigkeiten.  `fs-render` hat absichtlich einen minimalen
Dependency-Graph:

| Crate | Rolle |
|---|---|
| `std` (nur) | Keine externen Deps |

---

Siehe auch:
[fs-gui-engine-iced](fs-gui-engine-iced.md) (iced-Implementierung) ·
`fs-gui-engine-bevy` (Bevy / 3-D) ·
`fs-web-engine` (Browser-Engine-Abstraktion)
