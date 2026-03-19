# Cursor Manager — Mauszeiger-Sets verwalten & erstellen

> Lebt in **FreeSynergy.Managers** (`cursor/`, Crate: `fsn-manager-cursor`).

[← Zurück zum Index](../../INDEX.md) | [Icon Manager](README.md) | [Theme Manager](../theme/README.md) | [UI-Standards](../../konzepte/ui-standards.md) | [Synthesizer](../../konzepte/synthesizer.md)

---

## Was der Cursor Manager ist

Der Cursor Manager verwaltet alle installierten Mauszeiger-Sets und stellt eine UI bereit, um eigene Sets zu erstellen — manuell oder mit Unterstützung eines [Synthesizers](../../konzepte/synthesizer.md).

Er ist ein eigenständiger Menüpunkt innerhalb von **FreeSynergy.Managers**, mit derselben Sidebar-Struktur wie der Icon Manager.

---

## Sidebar-Struktur

```
┌─────────────────────────┐
│ [Aktives Set — Vorschau] │
├─────────────────────────┤
│ Installierte Sets        │  ← Liste + Vorschau + Aktivieren
│ Neues Set erstellen      │  ← Formular (manuell oder via Synthesizer)
│ ─────────────────────── │
│ Einstellungen (angepinnt)│  ← Repositories verwalten
└─────────────────────────┘
```

---

## Installierte Sets

Jedes Set wird mit einer Vorschau-Reihe aller Cursors angezeigt:
- Name, Autor, Version
- Vorschau: `default`, `pointer`, `text`, `busy`, `grab`, `not-allowed` (die wichtigsten 6)
- Button: **Aktivieren** (setzt das Set als aktives Theme-Cursor-Set)
- Button: **Bearbeiten** (öffnet das Formular mit den bestehenden Daten)
- Button: **Löschen** (nur wenn `builtin = false`)

Das aktuell aktive Set ist oben angepinnt und mit einem Marker gekennzeichnet.

---

## Neues Set erstellen

Das Formular deckt alle Metadaten und alle 31 Cursor-Slots ab.

### Metadaten-Felder

| Feld | Typ | Pflicht |
|---|---|---|
| `id` | Text (slug, lowercase, bindestriche) | ja |
| `name` | Text | ja |
| `description` | Textarea | nein |
| `author` | Text | nein |
| `version` | Text (SemVer, z.B. `1.0.0`) | ja |
| `animated` | Checkbox | nein |

### Cursor-Slots

Für jeden der 31 Standard-Cursor (siehe [UI-Standards](../../konzepte/ui-standards.md)) gibt es einen Slot:

```
[Cursor-Name]  [Vorschau]  [SVG hochladen]  [Hotspot X]  [Hotspot Y]
default        ●           [Datei wählen]   0             0
pointer        ↖           [Datei wählen]   6             0
text           I           [Datei wählen]   4             9
busy           ○           [Datei wählen]   0             0
...
```

- **Vorschau** wird sofort nach Upload angezeigt (SVG inline)
- **Hotspot** ist der aktive Pixel-Punkt (Standard: 0/0, Ausnahmen: `pointer`, `crosshair`, `grabbing`)
- Fehlende Slots: erlaubt — der CSS-Fallback greift automatisch

### Minimum-Pflicht

Das Formular warnt (kein Fehler, nur Hinweis) wenn weniger als diese 15 Slots ausgefüllt sind:
`default`, `pointer`, `not-allowed`, `busy`, `progress`, `text`, `move`, `grab`, `grabbing`, `drop-ok`, `drop-deny`, `resize-ns`, `resize-ew`, `resize-nwse`, `resize-nesw`

---

## Animierte Cursor

Wenn `animated = true` gesetzt ist, öffnet sich pro Cursor-Slot ein erweiterter Modus:

```
[Cursor-Name]  [Animation: ja/nein]
  Falls ja:
  Frame 1:  [SVG hochladen]  [Dauer: 80ms]
  Frame 2:  [SVG hochladen]  [Dauer: 80ms]
  Frame 3:  [SVG hochladen]  [Dauer: 80ms]
  [+ Frame hinzufügen]

  Vorschau: [▶ Abspielen]
```

Typischerweise werden nur `busy` und `progress` animiert. Alle anderen bleiben statisch.

### Export-Format für animierte Cursor

| Format | Wann |
|---|---|
| SVG-Frames (intern) | Speicherung in FSN — Frames als `busy-frame-1.svg`, `busy-frame-2.svg` etc. |
| APNG | Export für Webbrowser / Electron-Umgebungen |
| `.ani` / `.cur` | Export für Windows-Kompatibilität |

Das `manifest.toml` enthält bei animierten Cursors zusätzliche Felder:

```toml
[animated.busy]
frames     = ["busy-frame-1.svg", "busy-frame-2.svg", "busy-frame-3.svg"]
frame_ms   = [80, 80, 80]    # Dauer pro Frame in Millisekunden
loop       = true
```

---

## Synthesizer-Integration

Wenn ein Dienst mit Rolle `synthesizer.structured` installiert ist, erscheint im Formular-Header ein Button **"Mit Synthesizer erstellen"**.

→ Vollständige Beschreibung: [Synthesizer-Konzept](../../konzepte/synthesizer.md)

**Cursor-spezifisch:** Der Synthesizer bekommt als Kontext:
- Die vollständige Liste der erwarteten Cursor-Slots (inkl. semantischer Bedeutung)
- Die aktuelle Formular-Beschreibung (falls schon eingegeben)
- Optional: ein Referenz-Set (Stil übernehmen)

Der Synthesizer antwortet mit N Vorschlag-Sets — jeder Vorschlag enthält SVG-Code für alle 31 Cursors. Der Nutzer sieht eine Vorschau-Reihe pro Vorschlag und kann per Checkbox auswählen, welchen er ins Formular übernehmen möchte.

---

## Speichern & Veröffentlichen

Nach dem Ausfüllen des Formulars:

| Aktion | Voraussetzung |
|---|---|
| **Lokal speichern** | Immer möglich |
| **In Repository pushen** | Nutzer hat `cursor-set.publish`-Berechtigung auf dem Ziel-Repository |

Beim Pushen wird automatisch:
1. Das Set-Verzeichnis unter `cursor-sets/{id}/` angelegt
2. `manifest.toml` generiert und eingecheckt
3. Ein Git-Commit mit Autor-Info erstellt
4. Gepusht (wenn Berechtigung vorhanden)

---

## Einstellungen (angepinnt)

Identisch zum Icon Manager:
- Alle Cursor-Repositories auflisten
- Aktivieren / Deaktivieren
- Hinzufügen / Entfernen (nicht für `builtin`)

Das Haupt-Repository (`freesynergy-icons`) ist `builtin = true`.

---

## Verzeichnisstruktur

```
FreeSynergy.Icons/
├── manifest.toml               ← Sets + Cursor-Sets zusammen
├── cursor-sets/
│   ├── midnight/
│   │   ├── manifest.toml
│   │   ├── default.svg
│   │   ├── pointer.svg
│   │   ├── busy-frame-1.svg    ← animierter Cursor: Frame 1
│   │   ├── busy-frame-2.svg
│   │   ├── busy-frame-3.svg
│   │   └── ...
│   └── ...
└── ...
```

---

Weiter: [Icon Manager](README.md) | [Theme Manager](../theme/README.md) | [Synthesizer-Konzept](../../konzepte/synthesizer.md) | [UI-Standards](../../konzepte/ui-standards.md)
