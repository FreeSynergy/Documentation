# Desktop — Die Mensch-Maschine-Schnittstelle

[← Zurück zum Index](../../INDEX.md)

---

## Was Desktop macht

Desktop ist die UI für den Menschen. Es zeigt was Node und Container App Manager tun, bietet den Store-Browser, den Bot Manager, das Task-System, Widgets, Lenses und einen eingebetteten [Browser](../browser/README.md).

Desktop ist auch UI für die Services selbst — Web-Interfaces werden im eingebetteten Browser geöffnet.

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
| 🎛️ Container App Manager | Oben | Service-Konfiguration |
| 🤖 Bots | Oben | [BotManager](../botmanager/README.md) — Broadcasts senden, Gatekeeper verwalten, Status |
| 🔍 Lenses | Oben | Informations-Betrachter |
| 🌐 Browser | Oben | Eingebetteter Web-Browser für Service-UIs |
| 📦 Store | Oben | Pakete installieren |
| 🔎 Search | Oben | Suche über alle Services |
| 🔨 Builder | Oben | Resource Builder (Pakete bauen) |
| ⚙️ Settings | **Pinned** | Themes, Shortcuts, Profil, Layout |
| ❓ Help | Oben | Dokumentation |

## Interface-Typen

| Interface | Kürzel | Technologie |
|---|---|---|
| Web-based GUI | WGUI | Dioxus WebView (webkit2gtk / WebView2 / WKWebView) |
| Web | WGUI (WASM) | Dioxus WASM im Browser |
| Mobile | WGUI | Dioxus (Android/iOS — reiner Client) |
| Command Line | CLI | clap |

**WGUI** bedeutet: Die UI wird immer in einer Web-Engine gerendert (HTML/CSS). Auch auf dem Desktop ist es technisch ein WebView — kein natives GTK/Qt.

## UI-Objekt-System

Alle sichtbaren Elemente basieren auf dem [FsnObject-System](../../technik/ui-objekte.md).

## Eigenständigkeit

Desktop läuft auch offline. Für Live-Daten braucht es eine Verbindung zu einem Node.

## Plattformen

| OS | Status |
|---|---|
| Linux | ✅ (webkit2gtk) |
| macOS | ✅ (WKWebView) |
| Windows | ✅ (WebView2) |
| Android/iOS | ✅ Reiner Client (per Invite verbinden) |

## Repo

https://github.com/FreeSynergy/Desktop

## Bibliotheken

| Crate | Zweck |
|---|---|
| `dioxus` 0.7.x | UI-Framework (inkl. WebView für Browser) |
| `fsn-ui` | Komponenten-Bibliothek |
| `fsn-theme` | Theme-System |
| `fsn-i18n` | Sprach-Snippets |
| `fsn-db` | SQLite (fsn-desktop.db) |
| `fsn-store` | Store-Client |
| `reqwest` | Node-API aufrufen |

---

Weiter: [Browser](../browser/README.md) | [Lenses](../lenses/README.md) | [BotManager](../botmanager/README.md) | [UI-Objekte](../../technik/ui-objekte.md)
