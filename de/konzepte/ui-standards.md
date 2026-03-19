# UI-Standards — Mauszeiger, Icons, Menüpunkte

[← Zurück zum Index](../INDEX.md) | [Themes](themes.md) | [CSS-System](../technik/css.md) | [Icon Manager](../programme/icons/README.md)

---

## Prinzip: Klarheit und Austauschbarkeit

Namen beschreiben **was etwas ist**, nicht wie es heißt. Ein Browser heißt Browser — nicht Feuerfuchs oder Chromstern. Was hat ein Feuerfuchs mit einem Browser zu tun? Nichts. Was hat ein chromierter Stern mit einem Browser zu tun? Nichts. Wenn der Name erklärt was das Ding tut, braucht niemand eine Erklärung.

Dasselbe gilt für jeden Cursor, jedes Menüicon, jeden Zustand in der UI. **Der Name sagt was es bedeutet.** Niemand muss raten.

Der zweite Vorteil: Standardisierte Namen machen alles **austauschbar**. Wer ein Cursor-Set hochlädt, weiß genau welche Dateien erwartet werden. Wer ein Icon-Set erstellt, weiß genau welche Aktionen abgedeckt sein müssen. Ein Verzeichnis mit den richtigen Dateinamen ersetzt das gesamte Set — automatisch, ohne Konfiguration.

---

## Mauszeiger (Cursor Sets)

### Vollständige Cursor-Liste

Jeder Cursor hat einen **semantischen Namen** (= Dateiname im Set), eine Beschreibung **wann er erscheint**, und einen CSS-Fallback.

| Dateiname | Wann erscheint dieser Cursor | CSS-Fallback |
|---|---|---|
| `default` | Normaler Zustand, keine Interaktion | `default` |
| `pointer` | Klickbares Element (Button, Link, interaktives Widget) | `pointer` |
| `not-allowed` | Aktion nicht möglich (deaktiviert, keine Berechtigung, falscher Zustand) | `not-allowed` |
| `busy` | System arbeitet, Nutzer muss warten (Ladevorgang) | `wait` |
| `progress` | System arbeitet, Nutzer kann parallel weiter interagieren | `progress` |
| `text` | Text ist editierbar oder selektierbar | `text` |
| `text-vertical` | Vertikaler Text editierbar/selektierbar | `vertical-text` |
| `move` | Element kann verschoben werden (drag-fähiger Container) | `move` |
| `grab` | Element kann gegriffen werden (vor dem Drag) | `grab` |
| `grabbing` | Element wird gerade gezogen (während dem Drag) | `grabbing` |
| `crosshair` | Präzise Auswahl (Zuschneiden, Zeichnen, Messung) | `crosshair` |
| `zoom-in` | Klick vergrößert den Bereich | `zoom-in` |
| `zoom-out` | Klick verkleinert den Bereich | `zoom-out` |
| `help` | Zusätzliche Information verfügbar (Tooltip, Hilfe-Kontext) | `help` |
| `context-menu` | Rechtsklick öffnet ein Kontextmenü | `context-menu` |
| `alias` | Erstellt einen Verweis / eine Verknüpfung | `alias` |
| `cell` | Zellen-Auswahl (Tabelle, Spreadsheet) | `cell` |
| `drop-ok` | Drag-Element über einem gültigen Ziel | `copy` |
| `drop-deny` | Drag-Element über einem ungültigen Ziel | `no-drop` |
| `resize-n` | Größe nach oben ändern | `n-resize` |
| `resize-s` | Größe nach unten ändern | `s-resize` |
| `resize-e` | Größe nach rechts ändern | `e-resize` |
| `resize-w` | Größe nach links ändern | `w-resize` |
| `resize-ns` | Größe vertikal (bidirektional) | `ns-resize` |
| `resize-ew` | Größe horizontal (bidirektional) | `ew-resize` |
| `resize-ne` | Größe oben-rechts (Ecke) | `ne-resize` |
| `resize-nw` | Größe oben-links (Ecke) | `nw-resize` |
| `resize-se` | Größe unten-rechts (Ecke) | `se-resize` |
| `resize-sw` | Größe unten-links (Ecke) | `sw-resize` |
| `resize-nesw` | Größe diagonal bidirektional (oben-rechts ↔ unten-links) | `nesw-resize` |
| `resize-nwse` | Größe diagonal bidirektional (oben-links ↔ unten-rechts) | `nwse-resize` |

**Minimum-Set** (muss jedes Cursor-Set abdecken):
`default`, `pointer`, `not-allowed`, `busy`, `progress`, `text`, `move`, `grab`, `grabbing`, `drop-ok`, `drop-deny`, `resize-ns`, `resize-ew`, `resize-nwse`, `resize-nesw`

Alle anderen Cursor sind optional — fehlt eine Datei, greift der CSS-Fallback.

---

### Verzeichnisstruktur eines Cursor-Sets

```
cursor-sets/{set-id}/
├── manifest.toml
├── default.svg
├── pointer.svg
├── not-allowed.svg
├── busy.svg            ← animierbar: busy.png (Framestrip) oder busy.ani (Windows)
├── progress.svg
├── text.svg
├── text-vertical.svg
├── move.svg
├── grab.svg
├── grabbing.svg
├── crosshair.svg
├── zoom-in.svg
├── zoom-out.svg
├── help.svg
├── context-menu.svg
├── alias.svg
├── cell.svg
├── drop-ok.svg
├── drop-deny.svg
├── resize-n.svg
├── resize-s.svg
├── resize-e.svg
├── resize-w.svg
├── resize-ns.svg
├── resize-ew.svg
├── resize-ne.svg
├── resize-nw.svg
├── resize-se.svg
├── resize-sw.svg
├── resize-nesw.svg
└── resize-nwse.svg
```

### manifest.toml — Format

```toml
id          = "midnight-cursors"
name        = "Midnight Cursors"
description = "Cursors for the Midnight Blue theme"
author      = "FreeSynergy"
version     = "1.0.0"
builtin     = true

# Hotspot-Overrides (Pixel-Koordinate des aktiven Punkts, Default: 0/0)
[hotspots]
pointer    = [6, 0]
crosshair  = [12, 12]
grabbing   = [12, 8]
```

### Wie Cursor-Sets verwendet werden

Der **Theme Manager** verweist auf ein Cursor-Set per `cursor_set_id`. Beim Aktivieren eines Themes (oder beim manuellen Override in Settings → Appearance) werden alle Cursor aus dem Set geladen.

Das Set liegt entweder:
- im FreeSynergy.Icons-Repository (als Untertyp `cursor-set`)
- lokal in `shared/cursor-sets/{set-id}/`

Wer ein neues Set erstellen will: Verzeichnis mit den richtigen Dateinamen anlegen, `manifest.toml` hinzufügen, ins Repository hochladen. Kein Code nötig.

---

## Menüpunkte & Aktions-Icons

Menüpunkte brauchen ebenfalls standardisierte Icons — damit alle Programme dieselbe visuelle Sprache sprechen. **Ein Icon, ein Dateiname, ein Bedeutung.** Nie zwei verschiedene Icons für "Löschen" in zwei Programmen.

Zustände (hover, aktiv, deaktiviert) werden **nicht durch separate Dateien** abgebildet, sondern durch CSS (`opacity`, `color`, `filter`). Ein Icon = eine SVG-Datei.

### Standard-Aktions-Icons

**Bearbeitung:**

| Dateiname | Bedeutung |
|---|---|
| `add.svg` | Hinzufügen / Neu erstellen |
| `edit.svg` | Bearbeiten / Ändern |
| `delete.svg` | Löschen (unwiderruflich) |
| `remove.svg` | Entfernen aus Liste (nicht löschen) |
| `save.svg` | Speichern |
| `copy.svg` | Kopieren |
| `cut.svg` | Ausschneiden |
| `paste.svg` | Einfügen |
| `duplicate.svg` | Duplizieren |
| `undo.svg` | Rückgängig |
| `redo.svg` | Wiederholen |
| `clear.svg` | Leeren / Zurücksetzen |

**Navigation:**

| Dateiname | Bedeutung |
|---|---|
| `back.svg` | Zurück |
| `forward.svg` | Vorwärts |
| `up.svg` | Eine Ebene hoch |
| `home.svg` | Startseite / Übersicht |
| `menu.svg` | Menü öffnen (Hamburger) |
| `expand.svg` | Ausklappen |
| `collapse.svg` | Einklappen |
| `previous.svg` | Vorheriges Element |
| `next.svg` | Nächstes Element |
| `open.svg` | Öffnen / Navigieren zu |
| `close.svg` | Schließen / Dialog beenden |

**Daten & Transfer:**

| Dateiname | Bedeutung |
|---|---|
| `search.svg` | Suchen |
| `filter.svg` | Filtern |
| `sort.svg` | Sortieren |
| `upload.svg` | Hochladen |
| `download.svg` | Herunterladen |
| `import.svg` | Importieren |
| `export.svg` | Exportieren |
| `share.svg` | Teilen |
| `sync.svg` | Synchronisieren / Aktualisieren |
| `refresh.svg` | Neu laden |

**Status & Feedback:**

| Dateiname | Bedeutung |
|---|---|
| `check.svg` | Bestätigen / Fertig / Erfolgreich |
| `cancel.svg` | Abbrechen |
| `success.svg` | Erfolg |
| `warning.svg` | Warnung |
| `error.svg` | Fehler |
| `info.svg` | Information |
| `loading.svg` | Lädt (animierbar) |

**System & Einstellungen:**

| Dateiname | Bedeutung |
|---|---|
| `settings.svg` | Einstellungen |
| `profile.svg` | Profil / Benutzer |
| `sign-out.svg` | Abmelden |
| `help.svg` | Hilfe |
| `theme.svg` | Erscheinungsbild / Theme |
| `language.svg` | Sprache |
| `permissions.svg` | Berechtigungen |
| `key.svg` | Schlüssel / Authentifizierung |
| `network.svg` | Netzwerk / Verbindung |
| `notifications.svg` | Benachrichtigungen |

### Wo diese Icons herkommen

Die Standard-Aktions-Icons gehören zum **FreeSynergy.Icons**-Repository im Unterordner `FreeSynergy/`. Andere Icon-Sets (homarrlabs, simpleicons) decken Dienst-Icons ab — keine Aktionsicons.

Wer ein UI-Icon an einer anderen Stelle braucht: erst prüfen ob der Name schon existiert. Nie einen neuen Namen für eine bereits definierte Aktion erfinden.

---

## Allgemeine Namenskonvention

| Regel | Erklärung |
|---|---|
| Nur Kleinbuchstaben | `not-allowed.svg`, nicht `NotAllowed.svg` |
| Bindestriche statt Underscores | `zoom-in.svg`, nicht `zoom_in.svg` |
| Keine Versionsnummern im Dateinamen | `pointer.svg`, nicht `pointer-v2.svg` |
| Name = Bedeutung | Was das Icon/der Cursor **bedeutet**, nicht wie er aussieht |
| Kein Branding | `browser.svg`, nicht `firefox.svg` oder `chrome.svg` |
| Keine Abkürzungen | `settings.svg`, nicht `cfg.svg` |
| Englisch | Immer, auch in deutschsprachigen Projekten |

---

## Zusammenfassung

Jeder Cursor, jedes Menüicon hat **einen Namen** — und der Name beschreibt genau was er bedeutet. Nichts anderes. Wer ein neues Set erstellen möchte, nimmt dieses Dokument als Checkliste: alle Dateien nach der Liste benennen, Manifest hinzufügen, hochladen. Der Theme Manager tauscht alles automatisch aus.

Das ist Standardisierung: nicht weil es schön klingt, sondern weil es die einzige Methode ist, bei der nichts erklärt werden muss.

---

Weiter: [Themes](themes.md) | [CSS-System](../technik/css.md) | [Icon Manager](../programme/icons/README.md)
