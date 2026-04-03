# Komponenten-System (fs-render)

[← Zurück zum Index](../INDEX.md)

---

## Überblick

Das FreeSynergy Komponenten-System ermöglicht ein **deklaratives, engine-unabhängiges Desktop-Layout**.  
Komponenten beschreiben *was* angezeigt wird — die Engine entscheidet *wie*.

---

## Design Pattern: Interpreter

```
LayoutDescriptor (TOML-AST)
       │
       ▼
LayoutInterpreter::interpret()
       │
       ├── iced  → IcedLayoutInterpreter → Element<LayoutMessage>
       ├── TUI   → TuiLayoutInterpreter  → TuiLayoutOutput (ratatui Lines)
       └── Bevy  → BevyLayoutInterpreter → geplant (G2)
```

---

## Shell-Struktur

```
┌─────────────────────────────────────┐
│              Topbar                 │  top / fill / bottom slots
├────────┬────────────────────────────┤
│        │                            │
│Sidebar │       Main                 │  fill slot (expandiert)
│        │                            │
├────────┴────────────────────────────┤
│            Bottombar                │  bottom slot
└─────────────────────────────────────┘
```

Jeder Container hat drei Slot-Typen:

| Slot   | Verhalten |
|--------|-----------|
| `top`  | Stapelt von oben nach unten |
| `fill` | Teilt den verbleibenden Raum gleichmäßig |
| `bottom` | Stapelt von unten nach oben |

---

## LayoutDescriptor (TOML)

Jedes Paket deklariert seine Komponenten in `package.toml`:

```toml
[[component]]
id     = "inventory-list"
slot   = "fill"
wasm   = "components/inventory_list.wasm"
config = "components/inventory_list.toml"
```

Layout-Konfiguration (Desktop-Shell):

```toml
[topbar]
enabled = true
size    = 48

[sidebar]
enabled = true
size    = 240

[[sidebar.slots.fill]]
id   = "inventory-list"
slot = "fill"
wasm = "components/inventory_list.wasm"
```

---

## Schicht-Reihenfolge (Precedence)

```
Store-Default < Global (/etc/freesynergy/) < User (~/.config/freesynergy/)
```

Lade-Funktion: `LayoutDescriptor::load_layered(store, global, user)` — fehlende Schichten werden übersprungen.

---

## Speicherpfade (`StoragePaths`)

Jedes Paket kennt seine Pfade:

| Typ     | Pfad |
|---------|------|
| `user`  | `~/.local/share/freesynergy/{paket}` |
| `global`| `/var/lib/freesynergy/{paket}` |
| `config`| `~/.config/freesynergy/{paket}` |
| `cache` | `~/.cache/freesynergy/{paket}` |

---

## ComponentTrait

```rust
pub trait ComponentTrait: Send + Sync {
    fn component_id(&self) -> &str;          // z.B. "inventory-list"
    fn name_key(&self) -> &'static str;      // FTL-Key für den Namen
    fn description_key(&self) -> &'static str; // FTL-Key für Beschreibung
    fn slot_preference(&self) -> SlotKind;   // Standard: Fill
    fn min_width(&self) -> u32;              // für Responsive-Layout
    fn min_height(&self) -> u32;
    fn render(&self, ctx: &ComponentCtx) -> Vec<LayoutElement>;
}
```

---

## LayoutElement

Die Schnittmenge zwischen Komponente und Engine:

```rust
pub enum LayoutElement {
    Text { content, size, color },
    Button { label_key, action, style },
    Icon { name, size },
    Row { children, gap },
    Column { children, gap },
    List { items, scrollable },
    Separator { label_key },
    Badge { content, color },
    Spinner,
    Spacer { pixels },
}
```

---

## Hot-Reload

Der `HotReloadWatcher` überwacht Komponenten-Verzeichnisse per `inotify` (Linux):

```
~/.config/freesynergy/{paket}/components/*.toml → LayoutChanged
~/.config/freesynergy/{paket}/components/*.wasm → ComponentChanged
```

Die Desktop-Shell lauscht auf dem `Receiver<HotReloadEvent>` und lädt die betroffene Komponente neu — kein Neustart nötig.

---

## Standard-Komponenten

Werden beim Start der Desktop-Shell automatisch registriert (`register_standard_components`):

| ID | Slot | Datenquelle |
|----|------|-------------|
| `inventory-list` | Sidebar / fill | `fs-inventory` via gRPC |
| `pinned-apps` | Sidebar / bottom | `fs-session` via gRPC |
| `app-sections` | Main / fill | `fs-inventory` Kategorien |
| `system-info` | Bottombar / bottom | `fs-info` via gRPC |
| `notification-bell` | Topbar / top | `fs-bus` Notification-Topic |

---

## Engine-Implementierungen

### iced (fs-gui-engine-iced)

- **iced_aw 0.11**: `Spinner` (Loading-Animation), `Badge` (Zähler-Pills), `Card` (Komponenten-Panel)
- `IcedLayoutInterpreter::interpret()` → `Element<'static, LayoutMessage>`
- Alle String-Daten werden geklont → kein Lifetime-Problem

### ratatui / TUI (fs-gui-engine-tui)

- **tachyonfx 0.25**: `fx::fade_from()` — Fade-In beim ersten Erscheinen einer Komponente
- `TuiLayoutInterpreter::interpret()` → `TuiLayoutOutput` (Lines + Animations)
- `shell_rects()` / `center_rects()`: Ratatui-Layout-Helfer für die Shell-Zerlegung

### Bevy (geplant, G2)

- `bevy_tweening` für Tween-Animationen
- Wird in `fs-gui-engine-bevy` implementiert

---

## Sandbox-Regeln (O7)

Komponenten dürfen:
- **Lesen**: nur via gRPC zu FS-Services
- **Schreiben**: nur via Bus-Events
- **UI**: nur die eigene Slot-Area
- **Netzwerk**: kein direkter Zugriff

---

## Repos

| Repo | Inhalt |
|------|--------|
| `fs-render` | Traits + Structs + Standard-Komponenten |
| `fs-gui-engine-iced` | iced + iced_aw Interpreter |
| `fs-gui-engine-tui` | ratatui + tachyonfx Interpreter |
| `fs-gui-engine-bevy` | Bevy-Interpreter (geplant) |
| `fs-i18n` | `locales/{en,de}/render.ftl` |
