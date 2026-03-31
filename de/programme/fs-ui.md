# fs-ui

[← Zurück zum Index](../INDEX.md)

**Repo:** `FreeSynergy/fs-ui` · `/home/kal/Server/fs-ui/`
**Typ:** Library (kein Container)
**Capabilities:** keine (reine UI-Komponenten-Library)

---

## Was ist das?

`fs-ui` ist die Basiskomponenten-Library für FreeSynergy.
Enthält wiederverwendbare UI-Bausteine: Buttons, Cards, Forms, Modals, Tables, etc.

> **Migration geplant (G2):** Aktuell Dioxus-basiert. Migration zu `fs-render`/iced geplant,
> sobald G2 abgeschlossen ist.

---

## Komponenten

```
fs-ui/src/components/
├── button.rs        — Button + IconButton
├── card.rs          — Card-Container
├── form.rs          — Form + Input + Select
├── modal.rs         — Modal-Dialog
├── table.rs         — Data-Table
├── tabs.rs          — Tab-Navigation
├── sidebar.rs       — Navigations-Sidebar
├── notification.rs  — Notification-Banner
├── toast.rs         — Toast-Meldungen
├── spinner.rs       — Loading-Spinner
├── progress.rs      — Progress-Bar
├── badge.rs         — Status-Badge
├── tooltip.rs       — Tooltip
├── search_bar.rs    — Suchfeld
├── scroll_container.rs — Scrollbarer Bereich
├── context_menu.rs  — Kontext-Menü
├── code_block.rs    — Code-Darstellung
├── help_panel.rs    — Hilfe-Panel
├── lang_switcher.rs — Sprachumschalter
├── theme_switcher.rs— Theme-Umschalter
├── llm_chat.rs      — LLM-Chat-Interface
├── app_launcher.rs  — App-Launcher
├── status_bar.rs    — Status-Leiste
└── window.rs        — Fenster-Wrapper
```

---

## Migration zu fs-render (G2)

Alle Komponenten sollen `FsView`-Trait aus `fs-render` implementieren:

```rust
// Aktuell (Dioxus):
pub fn Button(cx: Scope<ButtonProps>) -> Element { ... }

// Ziel (fs-render):
impl FsView for Button {
    fn view(&self) -> Box<dyn FsWidget> { ... }
}
```

`view.rs` wird einziges Bindeglied — Domain-Objekte importieren `fs-render` nicht direkt.

---

## Dependencies

| Dependency | Zweck                                            |
|------------|--------------------------------------------------|
| Dioxus     | Aktuelles UI-Framework (Migration zu iced/G2)    |
| `fs-theme` | CSS-Variablen + Theming                          |
| `fs-i18n`  | Übersetzungen für Komponentenbeschriftungen       |
