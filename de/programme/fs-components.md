# fs-components

[← Zurück zum Index](../INDEX.md)

**Repo:** `FreeSynergy/fs-components` · `/home/kal/Server/fs-components/`
**Typ:** Library (kein Container)
**Capabilities:** keine (reine App-Komponenten-Library)

---

## Was ist das?

`fs-components` enthält wiederverwendbare App-Level-Komponenten — eine Ebene höher als `fs-ui`.
Während `fs-ui` generische Widgets bereitstellt (Button, Card, Form), bietet `fs-components`
fertige Bausteine für typische App-Bereiche (Nav, Chat, Toast, AppShell).

> **Migration geplant (G2):** Aktuell Dioxus-basiert (Feature `dioxus`).
> Migration zu `fs-render`/iced geplant.

---

## Zwei-Schicht-Architektur

```
Schicht 1 — Renderer-agnostig (immer kompiliert):
  toast_bus.rs    — Pub/Sub-Bus für Toast-Nachrichten (kein Dioxus nötig)

Schicht 2 — Dioxus-Komponenten (Feature "dioxus"):
  nav.rs          — NavItem, NavGroup, NavBar, NavBreadcrumb
  chat.rs         — Chat-Interface (für AI + Bots)
  app.rs          — AppShell (Desktop-Integration via fs-gui-workspace)
  form.rs / form_field.rs — Formulare + Felder
  button.rs       — Höherwertige Button-Varianten
  card.rs         — Content-Cards
  modal.rs        — Modals + Dialoge
  toast.rs        — Toast-Anzeige (konsumiert toast_bus)
  layout.rs       — Grid + Flex-Layout-Helper
  context.rs      — AppContext-Provider
  controls.rs     — Toolbar + Aktions-Buttons
  display.rs      — Text-Display + Code-View
  input.rs        — Erweiterte Eingabe-Felder
  launch.rs       — App-Start-Koordination
  overlay.rs      — Overlay-Layer
  styles.rs       — CSS-Variablen + Theme-Helper
```

---

## Bekannte Tech-Debt

```
nav.rs: ~1200 Zeilen — aufteilen in:
  → nav/item.rs + nav/group.rs + nav/bar.rs + nav/breadcrumb.rs
```

---

## Dependencies

| Dependency  | Zweck                                            |
|-------------|--------------------------------------------------|
| Dioxus      | Aktuelles UI-Framework (Migration zu iced/G2)    |
| `fs-ui`     | Basis-Widgets                                    |
| `fs-theme`  | CSS-Variablen + Theming                          |
| `fs-i18n`   | Übersetzungen                                    |
