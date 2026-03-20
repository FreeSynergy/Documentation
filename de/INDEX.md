# FreeSynergy — Dokumentation

**Sprache:** Deutsch  
**Stand:** März 2026  
**Repository:** https://github.com/FreeSynergy/Documentation

---

## Warum FreeSynergy?

Computer stellen Werkzeuge in den Vordergrund. Der Mensch muss wissen WELCHES Programm er öffnen muss, um an WELCHE Information zu kommen. Das ist rückwärts. Der Mensch braucht Informationen — und ob die aus einem Wiki, einer Datenbank, einer Karte oder einem Messenger kommen, ist irrelevant.

FreeSynergy dreht das um: **Die Information steht im Vordergrund, nicht das Werkzeug.** Programme arbeiten im Hintergrund zusammen. Der Mensch sieht nur das Ergebnis. Wenn er tiefer eintauchen will, klickt er sich zum Werkzeug durch — aber erst dann.

---

## Dokumentations-Übersicht

### Architektur
- [Gesamtübersicht](architektur/uebersicht.md) — Wie alles zusammenhängt

### Programme
- [Init](programme/init/README.md) — Der Bootstrap (installiert den Store)
- [Node](programme/node/README.md) — Der Projektverwalter + S3-Server
- [Container Manager](programme/container_app/README.md) — Container-Apps verwalten + Build & Publish (Builder ist hier integriert)
- [Builder](programme/builder/README.md) — **ARCHIVIERT** — Funktionalität jetzt im Container Manager
- [Language Manager](programme/language/README.md) — Sprache wechseln, Formate, Translation Editor, LLM-Assist
- [Theme Manager](programme/theme/README.md) — Themes, Farben, Mauszeiger, Fenster-Stil
- [Icon Manager](programme/icons/README.md) — Icon-Sets verwalten, Repository-Verwaltung, Icon-Picker für alle Programme
- [Cursor Manager](programme/icons/cursor-manager.md) — Mauszeiger-Sets verwalten, erstellen (manuell + Synthesizer), animierte Cursor
- [Icon-Set erstellen](programme/icons/icon-set-erstellen.md) — Formular: SVG-Upload, Dark-Varianten, Synthesizer-Integration, Veröffentlichen
- [Desktop](programme/desktop/README.md) — Die Mensch-Maschine-Schnittstelle
- [Browser](programme/browser/README.md) — Der eingebettete Web-Browser (eigenständig)
- [Store](programme/store/README.md) — Der Paketmanager & Das Wissen
- [Lenses](programme/lenses/README.md) — Der Informations-Betrachter
- [Search](programme/search/README.md) — Die mehrstufige Suche
- [Bot Manager](programme/botmanager/README.md) — Bot-Steuerung & Messenger-Integration, Control Bot (Accounts, Bots, Broadcast, Gatekeeper)

### Konzepte
- [Ressourcen-System](konzepte/ressourcen.md) — Alles ist eine Ressource (Structs, Typen, Felder)
- [Rollen-System](konzepte/rollen.md) — Services deklarieren was sie können
- [Bridges](konzepte/bridges.md) — Standardisierte Rollen-APIs
- [Inventory](konzepte/inventory.md) — Die lokale Wahrheit: **Store = möglich | Inventory = installiert | Managers = das Wie**
- [Rechte-Kaskade](konzepte/rechte.md) — Rechte können nur abnehmen
- [Message Bus](konzepte/bus.md) — Pub/Sub, Rollen-basiert, Bridges
- [Föderation](konzepte/foederation.md) — Zusammenarbeit zwischen Projekten
- [Pakete](konzepte/pakete.md) — Store-Paket-System: Kategorien (Server/App/Desktop), Typen, Versionierung
- [Node-Plattformen](konzepte/node-plattformen.md) — Linux-only Entscheidung, Szenarien (Pi, VPS, WSL2, macOS)
- [VPN & Netzwerk](konzepte/vpn.md) — WireGuard, Heimzugriff, Node-Mesh, das dezentrale Netz
- [Tasks](konzepte/tasks.md) — Automatisierungs-Pipelines
- [Bots](konzepte/bots.md) — Bot-Framework
- [Themes](konzepte/themes.md) — Visuelles Design-System
- [UI-Standards](konzepte/ui-standards.md) — Mauszeiger-Sets, Aktions-Icons, Menüpunkte: Naming Convention + Austauschbarkeit
- [Manager](konzepte/manager.md) — Kleber zwischen Store und Programmen (Language, Theme, ContainerApp, Icons)
- [Repository Manager](konzepte/repository-manager.md) — Gemeinsame Abstraktion für Repository-Verwaltung (Store, Icons, Bundles)
- [Synthesizer](konzepte/synthesizer.md) — Beschreibung → strukturierte Ausgabe: Formular-Vorausfüllung in allen Managern

### Technik
- [Storage-Layer (S3)](technik/storage.md) — Eigener S3-Server, opendal, Profiles
- [Datenspeicherung](technik/datenspeicherung.md) — SQLite-Architektur
- [Bibliotheken](technik/bibliotheken.md) — Welche Crates wofür
- [CSS-System](technik/css.md) — CSS-Variablen, Prefixing, Themes
- [UI-Objekt-System](technik/ui-objekte.md) — FsnObject: Fenster, Widgets, Modals
- [Installation](technik/installation.md) — Init → Store → Alles
- [Sicherheit](technik/sicherheit.md) — Encryption, Tokens, Signierung
- [i18n](technik/i18n.md) — Sprach-Snippets, RTL, Fallback

### TODO
- [Aktuelle TODO-Liste](todo/TODO.md) — Was gemacht werden muss
- [Bekannte Bugs](todo/BUGS.md) — Was kaputt ist
