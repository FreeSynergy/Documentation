# Bekannte Bugs

[← Zurück zum Index](../INDEX.md) | [TODO](TODO.md)

---

## Kritisch (🔴)

| Bug | Beschreibung | Fix |
|---|---|---|
| Window X-Button | Funktioniert nur im maximierten Zustand | Custom-Chrome fixen oder system decorations |
| Keine Speicherung | NICHTS wird persistent gespeichert — kein SQLite implementiert | Phase A1 |
| Store TOML Parse-Error | Catalog TOML hat Inline-Record-Fehler | Format fixen |

## Hoch (🟡)

| Bug | Beschreibung | Fix |
|---|---|---|
| Widgets nicht draggable | Können nicht per Drag & Drop verschoben werden | DraggableWidget (Phase D1) |
| Fenster/Widgets nicht resizable | Keine Resize-Handles an den Rändern | ResizableContainer (Phase D2) |
| Scrollbars fehlen | An vielen Stellen kein Scroll möglich | .fsn-scrollable überall (Phase A4) |
| Theme-Kontraste | Teilweise miserabler Kontrast, Text kaum lesbar | Theme-Überarbeitung (Phase B2) |

## Mittel (🟠)

| Bug | Beschreibung | Fix |
|---|---|---|
| Bilder nicht uploadbar | Kein Hintergrund-Upload möglich | FileEngine (Phase D4) |
| Store: Nichts installierbar | Install-Button funktioniert nicht | Phase C2-C4 |
| Alter Container App Manager-Code | Nutzt Podman-Socket (funktioniert nicht) | Phase E1 |

## Gering (🟢)

| Bug | Beschreibung | Fix |
|---|---|---|
| Leere Menüpunkte | Einige Menüpunkte ohne Funktion | Phase K4 |
| Stubs im Code | `todo!()`, `unimplemented!()` | Phase K9 |
