# Browser — Der eingebettete Web-Browser

[← Zurück zum Index](../../INDEX.md) | [Desktop](../desktop/README.md)

---

## Was der Browser macht

Der Browser ist eine App im Desktop die Web-Seiten anzeigt. Da Dioxus bereits WebView (webkit2gtk / WebView2 / WKWebView) integriert hat, reicht es eine URL zu übergeben.

## Warum ein eigener Browser?

1. **Service-UIs anzeigen** — Kanidm, Forgejo, Outline haben Web-Interfaces. Statt den System-Browser zu öffnen, zeigt der FreeSynergy-Browser sie direkt im Desktop an.
2. **Downloads** — Dateien aus Service-UIs herunterladen, direkt in den S3-Storage.
3. **Authentifizierung** — Der Browser kann automatisch IAM-Tokens mitschicken, kein separater Login nötig.
4. **Integration** — Links aus Lenses, Search, Widgets öffnen im Browser statt extern.

## Technisch

- Nutzt die vorhandene Dioxus WebView — kein extra Browser-Engine
- URL eingeben → Seite wird gerendert
- Download-Handler fängt Downloads ab → speichert in S3 `/shared/downloads/`
- Tabs für mehrere Seiten gleichzeitig
- Lesezeichen (in SQLite)
- History (in SQLite)

## Interfaces

| Interface | Funktion |
|---|---|
| URL-Leiste | Direkte URL-Eingabe |
| Tabs | Mehrere Seiten gleichzeitig |
| Service-Links | Klick auf Service im Container App Manager → öffnet dessen Web-UI |
| Lens-Links | Klick auf Ergebnis in Lens → öffnet im Browser |
| Downloads | Dateien → S3 Storage |

## Einschränkungen

- Kein vollständiger Browser (kein Erweiterungs-System, kein DevTools)
- Gedacht für Service-UIs und interne Seiten
- Für normales Surfen: System-Browser empfohlen

## Interfaces

| Interface | Beispiel |
|---|---|
| WGUI | Standalone-Fenster oder eingebettet in Desktop |
| CLI | `fsn browser open https://example.com` |
| API | `POST /api/browser/open { "url": "..." }` |

## Repo

https://github.com/FreeSynergy/Browser

Der Browser ist ein eigenständiges Programm. Desktop nutzt ihn als Dependency, aber er kann auch ohne Desktop gestartet werden.

---

Weiter: [Desktop](../desktop/README.md) | [Lenses](../lenses/README.md)
