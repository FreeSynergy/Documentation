# Theme Manager — Aussehen & Design

> Lebt in **FreeSynergy.Desktop** (`crates/fsd-theme/`). Eigenständiges Programm, im Desktop direkt aufrufbar.

[← Zurück zum Index](../../INDEX.md) | [Desktop](../desktop/README.md) | [CSS-System](../../technik/css.md)

---

## Bereiche

### Themes

Zeigt alle verfügbaren Themes (aus dem Store) als Karten an. Jedes Theme hat:
- Vorschau-Farbe
- Name
- "Aktivieren"-Button

Das aktive Theme wird mit einem "Active"-Badge markiert.

Themes kommen aus dem Store (Typ `theme`). Das eingebaute Standard-Theme ist **Midnight Blue**.

### Farben (Colors)

Zeigt alle CSS-Variablen des aktiven Themes in einer Liste:

| Variable | Farbe | Wert |
|---|---|---|
| `--fsn-color-primary` | 🟦 | `#00bcd4` |
| `--fsn-color-bg-base` | ⬛ | `#0f172a` |
| ... | | |

Jede Variable hat ein Farbvorschau-Kästchen und ein editierbares Textfeld mit dem Hex-Wert. Änderungen werden sofort auf die aktuelle Sitzung angewendet.

> **Speichern:** Die geänderten Farben gelten nur für die aktuelle Sitzung. Um sie dauerhaft zu speichern, kann man sie als eigenes Theme im Store veröffentlichen.

### Mauszeiger (Cursor)

Auswahl des Mauszeigers aus vier Optionen:
- **System Default** — Zeiger des Betriebssystems
- **FreeSynergy** — Angepasster Zeiger im FSN-Stil
- **Minimal** — Schlichter, kleiner Zeiger
- **Retro** — Klassischer Stil

### Fenster-Stil (Chrome)

Drei Einstellungsgruppen:

**Fenster-Stil (Window Chrome):**
- macOS — runde farbige Kreise (Ampel)
- KDE — rechteckige Buttons
- Windows — quadratische Buttons
- Minimal — kein Chrome

**Sidebar-Stil:**
- Solid — undurchsichtig
- Glass — Glasmorphismus (backdrop-filter)
- Transparent — vollständig transparent

**Animationen:**
- Ein / Aus — respektiert `prefers-reduced-motion`

---

## Repo

https://github.com/FreeSynergy/Desktop (Crate: `crates/fsd-theme/`)

---

Weiter: [Themes-Konzept](../../konzepte/themes.md) | [CSS-System](../../technik/css.md)
