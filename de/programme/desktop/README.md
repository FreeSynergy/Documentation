# Desktop — Die Mensch-Maschine-Schnittstelle

[← Zurück zum Index](../../INDEX.md)

---

## Was Desktop macht

Desktop ist die UI für den Menschen. Es zeigt was Node und Container App Manager tun, bietet den Store-Browser, den Bot Manager, das Task-System, Widgets, Lenses und einen eingebetteten [Browser](../browser/README.md).

Desktop ist auch UI für die Services selbst — Web-Interfaces werden im eingebetteten Browser geöffnet.

## Sidebar-Tabs

| Tab | Funktion |
|---|---|
| 🏠 Home | Persönliches Dashboard, Widgets, Quick-Launch |
| 📋 Tasks | Automatisierungs-Pipelines |
| 🎛️ Container App Manager | Service-Konfiguration |
| 🤖 Bots | [BotManager](../botmanager/README.md) — Broadcasts senden, Gatekeeper verwalten, Status |
| 🔍 Lenses | Informations-Betrachter |
| 🌐 Browser | Eingebetteter Web-Browser für Service-UIs |
| 📦 Store | Pakete installieren |
| 🔎 Search | Suche über alle Services |
| 🔨 Builder | Resource Builder (Pakete bauen) |
| ⚙️ Settings | Themes, Shortcuts, Profil, Layout |
| ❓ Help | Dokumentation |

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
