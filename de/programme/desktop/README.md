# Desktop — Die Mensch-Maschine-Schnittstelle

[← Zurück zum Index](../../INDEX.md)

---

## Was Desktop macht

Desktop ist die UI für den Menschen. Es zeigt was Node und Container Manager tun, bietet den Store-Browser, den Bot Manager, das Task-System, Widgets, Lenses und einen eingebetteten [Browser](../browser/README.md).

Desktop ist auch UI für die Services selbst — Web-Interfaces werden im eingebetteten Browser geöffnet.

## Rendering-Architektur

Desktop importiert **niemals** direkt eine GUI-Bibliothek (iced, bevy o.ä.).
Stattdessen nutzt es die Abstraktionsschicht [fs-render](../../architektur/render-architektur.md):

```
fs-desktop / fs-apps
      ↓ (nur Traits)
  fs-render  ←→  fs-theme
      ↓
fs-gui-engine-iced   (Standard — libcosmic)
fs-gui-engine-bevy   (Optional — 3D-fähig)
```

Die Engine wird per Feature-Flag oder Laufzeit-Konfiguration gewählt.
Ein Wechsel der Engine erfordert keine Änderung am App-Code.

## Sideboard (Sidebar)

Die Sidebar — das **Sideboard** — ist die primäre Navigation des Desktops. Es ist in zwei Bereiche aufgeteilt:

### Oberer Bereich (scrollbar)

Enthält die regulären Navigationspunkte. Wenn zu viele Einträge vorhanden sind, wird dieser Bereich **scrollbar** — der untere Bereich bleibt davon unberührt.

### Unterer Bereich (Pinned Section — immer sichtbar)

Der untere Bereich ist **immer sichtbar**, egal wie viele Einträge im oberen Bereich stehen. Er enthält dauerhaft angepinnte Punkte (z.B. Einstellungen). Dieser Bereich scrollt nie weg.

### Ordner-Struktur

Im Sideboard können **Ordner** angelegt werden. Ordner gehören immer zu dem Bereich, in dem sie sich befinden:
- Ordner im **oberen Bereich** → Unterelemente erscheinen im oberen Bereich
- Ordner in der **Pinned Section** → Unterelemente erscheinen im unteren Bereich

Wenn man einen Ordner öffnet, ersetzt sein Inhalt die normale Liste im jeweiligen Bereich. Das **erste Element** ist immer ein **Pfeil nach hinten** (← Zurück), um zur vorherigen Ansicht zu gelangen.

**Besonderheit:** Hat ein Ordner nur **ein einziges Element**, wird kein Ordner angezeigt — der Inhalt erscheint direkt als normaler Eintrag.

### Installations-Orte aus Paketen

Pakete definieren selbst, **wo** sie im Sideboard erscheinen (oberer Bereich, Pinned Section, in welchem Ordner). Diese Struktur wird direkt aus den Paket-Metadaten übernommen und in der UI so dargestellt. Kein manuelles Sortieren notwendig.

### Sidebar-Tabs

| Tab | Bereich | Funktion |
|---|---|---|
| 🏠 Home | Oben | Persönliches Dashboard, Widgets, Quick-Launch |
| 📋 Tasks | Oben | Automatisierungs-Pipelines |
| 🎛️ Container Manager | Oben | Service-Konfiguration |
| 🤖 Bots | Oben | [BotManager](../botmanager/README.md) — Broadcasts senden, Gatekeeper verwalten, Status |
| 🔍 Lenses | Oben | Informations-Betrachter |
| 🌐 Browser | Oben | Eingebetteter Web-Browser für Service-UIs |
| 📦 Store | Oben | Pakete installieren — 3 Sections: Server, Apps, Desktop |
| 🔎 Search | Oben | Suche über alle Services |
| 🔨 Builder | Oben | Resource Builder (Pakete bauen) |
| ⚙️ Settings | **Pinned** | Themes, Shortcuts, Profil, Layout, Animationen |
| ❓ Help | Oben | Dokumentation |

## Interface-Typen

| Interface | Kürzel | Technologie |
|---|---|---|
| Native GUI | GUI | fs-render → fs-gui-engine-iced (Standard) oder fs-gui-engine-bevy |
| Web (WASM) | WASM | fs-render → WASM-Target (geplant) |
| Mobile | GUI | fs-render → Mobile-Engine (geplant) |
| Command Line | CLI | clap |

**GUI** bedeutet: Die UI wird über `fs-render` gerendert — welche Engine dahinter liegt, ist konfigurierbar.
Der Desktop läuft nativ — kein WebView, kein Chromium, keine Browser-Engine für die Haupt-UI.

## Konfigurierbare Desktop-Einstellungen

Der Desktop ist hochgradig konfigurierbar — angelehnt an KDE:

| Bereich | Was konfiguriert werden kann |
|---|---|
| Fensterverhalten | Titelleiste-Stil, Resize-Edges, Snap-Zones, Doppelklick-Aktion |
| Click-Verhalten | Single/Double-Click-Aktionen, Focus-Follows-Mouse, Drag-Threshold |
| Animationen | AnimationSet wählen, Geschwindigkeit, einzelne Animationen deaktivieren |
| Theme | Farben, Fonts, Abstände, Rounding, Schatten (→ fs-theme) |
| Icon-Sets | Icon-Set wählen (→ fs-icons) |
| Cursor-Sets | Mauszeiger-Set wählen (→ fs-icons) |
| Shortcuts | Alle Tastenkürzel konfigurierbar (Action Registry) |
| Workspace | Sideboard-Position, Panel-Anordnung, Spalten-Layout |

## Animations-System

Animationen sind **Store-Pakete** — austauschbar und erweiterbar wie Icon-Sets:

- **Vordefiniert:** slide-left, slide-right, fade, scale, rotate, ...
- **User-Animationen:** WASM-basiert via `fs-plugin-sdk` — eigene Timing-Curves
- **Konfigurierbar:** je App-Aktion welche Animation + Geschwindigkeit oder aus
- **Store-erweiterbar:** Community-Animations-Packs im Store veröffentlichbar

## UI-Objekt-System

Alle sichtbaren Elemente basieren auf dem [FsnObject-System](../../technik/ui-objekte.md).

## Eigenständigkeit

Desktop läuft auch offline. Für Live-Daten braucht es eine Verbindung zu einem Node.

## Plattformen

| OS | Status |
|---|---|
| Linux | ✅ (primär, fs-gui-engine-iced via libcosmic) |
| macOS | ✅ (fs-gui-engine-iced) |
| Windows | ✅ (fs-gui-engine-iced) |
| Android/iOS | Geplant (reiner Client — per Invite verbinden) |

## Repos

- **Shell:** https://github.com/FreeSynergy/fs-desktop
- **Apps:** https://github.com/FreeSynergy/fs-apps
- **Render-Abstraktion:** https://github.com/FreeSynergy/fs-render
- **GUI-Engine iced:** https://github.com/FreeSynergy/fs-gui-engine-iced
- **GUI-Engine Bevy:** https://github.com/FreeSynergy/fs-gui-engine-bevy

## Desktop-Crates (fs-desktop/crates/) — Shell & Infrastruktur

| Crate | Zweck |
|---|---|
| `fs-shell` | Desktop-Shell: Taskbar, Fenstermanager, Sidebar, WindowFrame |
| `fs-settings` | Einstellungen: Appearance, Language, Rollen, Desktop, Pakete, Animationen |
| `fs-profile` | Benutzerprofil |
| `fs-db-desktop` | SQLite-Schemas für Desktop + Apps |
| `fs-app` | Haupt-Launcher-Binary |
| `fs-showcase` | Komponenten-Galerie (nur Debug-Builds) |

## App-Crates (fs-apps/crates/) — Eigenständige Apps

| Crate | Zweck |
|---|---|
| `fs-store-app` | Paket-Browser (Store-UI) |
| `fs-browser` | Eingebetteter Web-Browser (nutzt fs-web-engine) |
| `fs-lenses` | Aggregierte Cross-Service-Ansichten |
| `fs-theme-app` | Theme-Manager (Farben, Cursor, Chrome) |
| `fs-builder` | Container-Builder, Bridge-Builder, i18n-Editor |
| `fs-tasks` | Task-Manager und Pipeline-Editor |
| `fs-bots` | Bot-Management-UI |
| `fs-ai` | KI-Assistent |
| `fs-container-app` | Container/Service-Verwaltung |
| `fs-managers` | Vereintes Manager-Panel (Language, Icons, Cursor, Theme, Container) |

## Bibliotheken

| Crate | Zweck |
|---|---|
| `fs-render` | GUI-Abstraktions-Traits — einziger Render-Einstiegspunkt |
| `fs-gui-engine-iced` | iced-Engine (Standard), libcosmic-Basis |
| `fs-gui-engine-bevy` | Bevy-Engine (3D, optional) |
| `fs-theme` | Theme-System |
| `fs-i18n` | Sprach-Snippets (Mozilla Fluent) + Locale-Formatierung |
| `fs-db` | SQLite (fs-desktop.db) |
| `fs-pkg` | Paket-Typen: Manageable-Trait, Package-Objekt, ConfigField |
| `fs-store` | Store-Client |
| `reqwest` | Node-API aufrufen |

## OOP-Prinzipien

- **`Manageable`-Trait:** Jedes Paket beschreibt sich selbst — Status, Felder, Health (→ [Pakete](../../konzepte/pakete.md#manageable-trait))
- **`FsWindow`-Trait:** Jedes Fenster implementiert FsWindow — einheitliche Titelleiste, konfigurierbare Panels
- **`WindowLayout`:** Sidebar/Hilfe-Panel Position per Nutzer konfigurierbar
- **`ManagerView`:** Eine Komponente für alle Pakete — das Paket liefert das WAS, der Manager das WIE
- **`PackageSettingsView`:** Alle Paket-Einstellungen aggregiert an einem Ort
- **`RenderEngine`-Trait:** Desktop kennt keine Engine — nur den Trait (→ [Render-Architektur](../../architektur/render-architektur.md))

---

Weiter: [Render-Architektur](../../architektur/render-architektur.md) | [Browser](../browser/README.md) | [Lenses](../lenses/README.md) | [BotManager](../botmanager/README.md) | [Manager](../../konzepte/manager.md)
