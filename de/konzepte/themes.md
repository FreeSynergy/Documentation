# Themes

[← Zurück zum Index](../INDEX.md) | [CSS-System](../technik/css.md)

---

## Was ein Theme definiert

Ein Theme definiert nicht nur Farben, sondern auch:
- **Window-Chrome:** macOS-Kreise, KDE-Rechtecke, Windows-Quadrate, Minimal
- **Button-Style:** Rounded, Square, Pill, Flat
- **Sidebar-Style:** Solid, Glass, Transparent
- **Card-Style:** Bordered, Shadow, Glass, Flat
- **Input-Style:** Underline, Bordered, Filled
- **Animationen:** Dauer, Easing, aktiviert/deaktiviert
- **Mauszeiger:** Standard, Custom-URL

## Eingebautes Theme vs. Store

**Eingebaut (immer vorhanden):** Nur **Midnight Blue** ist direkt in FreeSynergy.Desktop eingebaut.

**Via Store:** Alle anderen Themes kommen aus dem Store (Typ `theme`). Sie können heruntergeladen, installiert und live gewechselt werden — ohne Neustart.

Mitgelieferte Store-Themes (im FreeSynergy.Node.Store vorinstalliert):
- **Cloud White** — helles, klares Theme
- **Cupertino** — macOS-inspiriert, hell
- **Nordic** — Nord-Palette, dunkel
- **Rose Pine** — warme Rosetöne, dunkel

## Einzelaspekte vs. komplettes Theme

Store-Pakete sind immer **komplette Themes** (alle Aspekte definiert). Einzelne Aspekte lassen
sich jedoch in den Settings **unabhängig überschreiben** — ohne das Theme zu wechseln:

| Aspekt | CSS-Variable | Einstellungsort |
|---|---|---|
| Window Chrome | `--fsn-chrome-style` | Settings → Appearance → Component Style |
| Button Style | `--fsn-btn-radius` | Settings → Appearance → Component Style |
| Sidebar Style | `--fsn-sidebar-style` | Settings → Appearance → Component Style |
| Card Style | `--fsn-card-style` | Settings → Appearance → Component Style |
| Input Style | `--fsn-input-style` | Settings → Appearance → Component Style |
| Cursor | CSS `cursor` auf `:root` | Settings → Appearance → Component Style |
| Animationen | `--fsn-transition`, `--fsn-anim-enabled` | Settings → Appearance → Window Chrome |

Diese Overrides werden in `fsn-desktop.db` gespeichert und gelten über Theme-Wechsel hinaus.

## Theme-Paket-Struktur (Store)

```
shared/themes/{theme-id}/
├── manifest.toml   ← PackageMeta + [theme.colors], [theme.effects], [theme.typography]
└── icon.svg        ← Farbige Vorschau (SVG)
```

CSS-Variablen im Store: **ohne Prefix** (z.B. `--bg-base`). Beim Laden durch das Programm
wird der Prefix automatisch ergänzt (Desktop: `--fsn-`, Wiki.rs: `--wiki-`).

## Custom Theme Editor

In Settings → Appearance gibt es einen Theme-Editor: CSS mit unprefixed Variablen einfügen,
Validierung prüft Pflicht-Variablen, `--fsn-` Prefix wird automatisch gesetzt.

Weiter: [CSS-System](../technik/css.md) | [UI-Objekte](../technik/ui-objekte.md)
