# FreeSynergy — Dokumentation

**Sprache:** Deutsch
**Stand:** März 2026
**Repository:** https://github.com/FreeSynergy/fs-documentation

---

## Warum FreeSynergy?

Computer stellen Werkzeuge in den Vordergrund. Der Mensch muss wissen WELCHES Programm
er öffnen muss, um an WELCHE Information zu kommen. Das ist rückwärts.

FreeSynergy dreht das um: **Die Information steht im Vordergrund, nicht das Werkzeug.**
Programme arbeiten im Hintergrund zusammen. Der Mensch sieht nur das Ergebnis.
Wenn er tiefer eintauchen will, klickt er sich zum Werkzeug durch — aber erst dann.

---

## Dokumentations-Übersicht

### Architektur
- [Gesamtübersicht](architektur/uebersicht.md) — Philosophie, Schichten-Diagramm, Programme, Datenfluss
- [Repository-Übersicht](architektur/repositories.md) — Jedes Repo, sein Zweck, lokaler Pfad, GitHub-URL
- [Render-Architektur](architektur/render-architektur.md) — GUI-Abstraktion (fs-render), Engines (iced, Bevy), Browser-Engine (Servo), Animations-System
- [fs-render](programme/fs-render.md) — GUI-Abstraktions-Traits: RenderEngine, FsView, FsWidget, FsWindow, FsTheme, FsEvent, AppContext, AnimationRegistry, Fs3dExtension
- [fs-gui-engine-iced](programme/fs-gui-engine-iced.md) — iced Render-Engine: IcedEngine, MVU-Muster, IcedTheme, Capability "render.engine.iced"
- [fs-session](programme/fs-session.md) — Session-Management: SessionStore-Trait, SqliteSessionStore, SessionTracker, AppSession
- [fs-info](programme/fs-info.md) — System-Info: SystemInfo-Trait, FsInfo-Facade, MetricsCollector, Alerting
- [fs-container](programme/fs-container.md) — Container-Management: ContainerEngine-Trait, QuadletManager, ServiceConfig (Podman Quadlet)

### Programme
- [Init](programme/init/README.md) — Bootstrap: prüft System, lädt Store
- [Node](programme/node/README.md) — Server: Projekte, Hosts, S3, Auth, Federation, externer Zugriff
- [Desktop](programme/desktop/README.md) — Mensch-Maschine-Schnittstelle: Shell, Fenster, Widgets
- [Store](programme/store/README.md) — Paketmanager: Katalog lesen, Pakete installieren, Versionen verwalten
- [Browser](programme/browser/README.md) — Eingebetteter Web-Browser (eigenständig, kein Node nötig)
- [Lenses](programme/lenses/README.md) — Informations-Betrachter: Daten aus mehreren Services aggregiert
- [Tasks](programme/tasks/README.md) — Automatisierungs-Pipeline-Editor: Data Offers/Accepts, visuelle Verknüpfung
- [Search](programme/search/README.md) — Mehrstufige Suche: lokal → Host → föderiert
- [AI](programme/ai/README.md) — Lokaler LLM-Manager (Mistral.rs, Qwen-Modelle, Editor-Integration)
- [Bot Manager](programme/botmanager/README.md) — Bots konfigurieren, mit Messengern verbinden, Status überwachen
- [Container Manager](programme/container/README.md) — Container-Apps installieren, konfigurieren, starten/stoppen
- [Language Manager](programme/language/README.md) — Sprache wechseln, Formate, Übersetzungs-Editor
- [Theme Manager](programme/theme/README.md) — Themes, Farben, Mauszeiger, Fenster-Stil
- [Icon Manager](programme/icons/README.md) — Icon-Sets verwalten, Icon-Picker
- [Cursor Manager](programme/icons/cursor-manager.md) — Mauszeiger-Sets verwalten und erstellen
- [Icon-Set erstellen](programme/icons/icon-set-erstellen.md) — SVG-Upload, Dark-Varianten, veröffentlichen

> Alle Manager leben im Repo `FreeSynergy/fs-managers` als gemeinsamer Workspace.
> Builder ist nicht mehr eigenständig — Funktionalität ist im Container Manager aufgegangen.

### Konzepte
- [Pakete](konzepte/pakete.md) — Paket-Typen, `[[binaries]]`, Capabilities, Manifest-Format, Lifecycle
- [Inventory](konzepte/inventory.md) — Vier Ebenen: Store / Inventory / Registry / Managers
- [Registry](konzepte/registry.md) — Welche Services laufen gerade (Capabilities, Endpoints, Adapter-Lookup)
- [Adapter-Pattern](konzepte/adapter.md) — Drittanbieter-Dienste einbinden via Standard-Traits (ersetzt Bridge)
- [Message Bus](konzepte/bus.md) — Pub/Sub, Topic-Routing, kein direkter Service-Aufruf
- [Bus-API Namespaces](konzepte/bus-api-namespaces.md) — Standardisierte Topic-Adressen, Vertragsregeln
- [Rollen-System](konzepte/rollen.md) — Capabilities deklarieren (provides/requires), Hierarchie
- [Capabilities](konzepte/capabilities.md) — Was Pakete anbieten und was sie brauchen
- [Ressourcen-System](konzepte/ressourcen.md) — Alles ist eine Ressource: Structs, Typen, Felder
- [Session](konzepte/session.md) — Aktiver User, offene Programme, Minimize/Restore
- [Rechte-Kaskade](konzepte/rechte.md) — Rechte können nur abnehmen, nie erweitern
- [Föderation](konzepte/foederation.md) — Zusammenarbeit zwischen Nodes (opt-in, dezentral)
- [Node-Plattformen](konzepte/node-plattformen.md) — Linux-only Entscheidung, Szenarien (Pi, VPS, WSL2)
- [VPN & Netzwerk](konzepte/vpn.md) — WireGuard, Node-Mesh, Heimzugriff
- [SysInfo](konzepte/sysinfo.md) — Platform-Detection, dynamische Daten, Alerting
- [Tasks](konzepte/tasks.md) — Automatisierungs-Pipelines, Data Offers/Accepts
- [Bots](konzepte/bots.md) — Bot-Framework, Messenger-Adapter, mehrere Binaries pro Paket
- [Themes](konzepte/themes.md) — Visuelles Design-System, Theme-Ressourcen
- [UI-Standards](konzepte/ui-standards.md) — Naming Convention, Aktions-Icons, Menüpunkte
- [Manager](konzepte/manager.md) — Konfigurationswerkzeuge für alle Paket-Kategorien (7 Manager)
- [Repository Manager](konzepte/repository-manager.md) — Gemeinsame Abstraktion für Repo-Verwaltung
- [Synthesizer](konzepte/synthesizer.md) — Beschreibung → strukturierte Ausgabe, Formular-Vorausfüllung
- [AI-Inferenz](konzepte/ai-inferenz.md) — Lokale LLMs via Mistral.rs, Modelle, Workflow
- [Bridges](konzepte/bridges.md) — **ARCHIVIERT** — ersetzt durch Adapter-Pattern (März 2026)

### Technik
- [Build-Workflow](technik/build-workflow.md) — Design Pattern → OOP → Tests → Clippy → Push, Quality Gates, Binary-Distribution
- [Storage-Layer (S3)](technik/storage.md) — Eigener S3-Server, opendal, Profiles
- [Datenspeicherung](technik/datenspeicherung.md) — SQLite-Architektur, Datenbank pro Service
- [fs-db-engine-sqlite](technik/fs-db-engine-sqlite.md) — SQLite-Adapter: DbEngine-Trait-Impl, gRPC+REST, Capability "db.engine.sqlite"
- [fs-db-engine-postgres](technik/fs-db-engine-postgres.md) — PostgreSQL-Adapter: DbEngine-Trait-Impl, gRPC+REST, Capability "db.engine.postgres"
- [Bibliotheken](technik/bibliotheken.md) — Welche Crates wofür
- [Typen-System](technik/typen.md) — FsValue, FsUrl, LanguageCode, SemVer, FsPort, FsTag
- [CSS-System](technik/css.md) — CSS-Variablen, Prefixing, Themes
- [UI-Objekt-System](technik/ui-objekte.md) — Fenster, Widgets, Modals
- [Installation](technik/installation.md) — Init → Store → Alles
- [Sicherheit](technik/sicherheit.md) — Encryption, Tokens, Signierung
- [i18n](technik/i18n.md) — Mozilla Fluent, Sprach-Snippets, RTL, Fallback

### TODO
- [Build Plan](todo/TODO.md) — Aktive Phasen, nächste Gespräche, Archiv / Ideen-Pool
