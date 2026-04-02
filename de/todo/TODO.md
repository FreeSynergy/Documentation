# FreeSynergy — Build Plan

[← Zurück zum Index](../INDEX.md)

---

## Regeln für Claude-Sessions

- TODO vollständig lesen zu Beginn jeder Session
- Erledigte Items löschen — kein Erledigtes behalten, spart Token
- Vor jeder Entscheidung fragen — nie eigenmächtig handeln
- Repo-Arbeit immer: erst lesen → dann planen → dann umsetzen

---

## Status-Markierungen

```
[ ]  Muss noch gemacht werden
[o]  Wird gerade bearbeitet
[~]  Muss noch getestet werden
```

---

## Standard-Checkliste — gilt für JEDE Aufgabe

Jede Phase / jeder Task wird in dieser Reihenfolge bearbeitet.
Kein Task gilt als erledigt, bevor alle Punkte erfüllt sind:

```
1. Design Pattern festlegen (Strategy / MVC / Command / Observer / ...)
   → Entscheidung ZUERST, BEVOR Code geschrieben wird

2. OOP-Struktur definieren
   → Traits + Structs zuerst, dann Impl
   → Traits statt match-Blöcke
   → Objekte statt reine Daten
   → Immer gegen Interface (Trait) — nie gegen konkrete Impl

3. i18n von Anfang an
   → Jeder user-facing Text bekommt einen FTL-Key
   → KEINE Ausnahme — auch nicht "erstmal hardcoded"
   → KEINE Stubs oder leere Implementierungen — immer konkret
   → keys.rs mit FTL-Konstanten anlegen
   → .ftl-Datei in fs-i18n/locales/{lang}/{programm}.ftl

4. Kompilierbarkeit sicherstellen
   → cargo build muss nach jeder Änderung grün sein
   → Kein "wird später fixen" — sofort beheben

5. Abschluss-Check (exakt wie im Git-Hook):
   → cargo fmt --check (gleiche Konfiguration wie pre-commit)
   → cargo clippy (gleiche Flags wie pre-commit — kein Abweichen)
   → cargo test (alle Tests grün)
   → Erst wenn alle drei grün: Phase / Task abgeschlossen
```

**Wichtig zu clippy/fmt:** Die Einstellungen im Git-Hook sind maßgeblich.
Claude verwendet dieselben Flags wie der Hook — nie abweichende Konfigurationen,
um Fehler beim Commit zu vermeiden.

---

## Architektur-Überblick (Stand 2026-04-02)

### Grundprinzipien

```
Standalone-First:
  Jedes Repo / jeder Container funktioniert unabhängig vom Rest.
  Integration über gRPC / REST / Bus — nie harte Runtime-Dependencies.

Everything through the Store:
  Was nicht im Store ist, existiert für das System nicht.
  Pakete, Engines, Themes, Sprachen, Adapter — alles ist ein Store-Artifact.

Engine-Auswahl zur Laufzeit (entschieden 2026-04-02):
  fs-info ermittelt Capabilities (display_server, terminal, headless).
  fs-init zeigt anfangs NUR println! — keine zusätzlichen Libraries.
  Die Render-Engine wird danach als Artifact aus dem Store geladen.
  display_server available → GUI (iced oder bevy — User wählt)
  nur terminal             → TUI
  headless / SSH           → API + CLI only

Engine-Wahl im Bundle (entschieden 2026-04-02):
  Bundles mit UI-Anforderung enthalten "render-engine" als required-Artifact.
  Beim Install: User wählt aus verfügbaren Engines (Select / Dropdown).
  Standard-Vorschlag: iced (wenn display_server vorhanden), TUI sonst.
  Keine Engine ist hartgekodiert.

Adapter kommt immer mit:
  Jedes program-Paket hat einen adapter als Dependency.
  Store installiert beides zusammen.
  Adapter registriert Capability in fs-registry.

Deklarative UI-Abstraktion (entschieden 2026-04-02):
  Layout-Format: TOML (pro Komponente eine eigene .toml-Datei).
  Hot-Reload via inotify (wie FTL in fs-i18n).
  Komponenten als WASM-Plugins (fs-plugin-sdk / fs-plugin-runtime).
  Vorab kompiliert + gecheckt — dynamisch geladen zur Laufzeit.
  Desktop liefert Basis-Komponenten.
  Andere Programme können eigene Komponenten/Widgets mitbringen.
  ComponentId-Registry: Komponente meldet sich beim Start an.
  Engine (iced/bevy/TUI) interpretiert Layout eigenständig.
```

### Install-Kette

```
fs-init
  1. fs-info: ermittelt Capabilities (display? terminal? headless?)
  2. Ausgabe: nur println! — keine Bibliotheken außer fs-info
  3. Lädt Store-Service
  4. Store zeigt Bundles + Einzelpakete
  5. User wählt Render-Engine aus verfügbaren (Select in Bundle)
  6. User wählt Install-Target: Container (bevorzugt) | rpm | deb | flatpak | AppImage
  7. fs-info schlägt passende Variante vor (OS-Detection)
  8. Adapter wird IMMER mitinstalliert (nicht optional)
  9. Manager übernimmt Konfiguration nach Install
```

### Package-Typen im Store

```
bundle   — Gruppe von Paketen (z.B. Workstation, Server, Minimal)
program  — Laufender Dienst (Container bevorzugt)
adapter  — Implementiert Trait, registriert Capability (kommt mit program)
artifact — Reine Daten (Sprachen, Themes, DB-Engines, Render-Engines, Bots, WASM-Komponenten)
fork     — 3rd-party Fork (Kanidm, Tuwunel, Stalwart, Zentinel, ...)
```

### Deklaratives Komponenten-System (fs-render, entschieden 2026-04-02)

```
Format: TOML — pro Komponente eine eigene Datei (hotpluggable)
Hot-Reload: inotify überwacht Komponenten-Verzeichnis
Implementierung: WASM-Plugins via fs-plugin-sdk + fs-plugin-runtime
  → Vorab kompiliert + gecheckt (Compile-Time-Sicherheit)
  → Dynamisch geladen zur Laufzeit (kein Neustart nötig)

Shell-Container: Sidebar | Topbar | Bottombar | Main
Slots pro Container: top | fill | bottom
  → mehrere top-Komponenten stacken
  → fill-Komponenten teilen den verbleibenden Raum
  → mehrere bottom-Komponenten stacken

Komponente kennt ihre Mindest-/Maxgröße (responsive).
Neue Komponenten kommen als Store-Artifacts (WASM + TOML-Config).
```

### Bundle-Definitionen (entschieden 2026-04-02)

```
Minimal:     fs-init + fs-store + fs-registry + fs-inventory
Server:      Minimal + fs-auth(Kanidm) + fs-node + Zentinel + Stalwart
Workstation: Server + Desktop + render-engine(User-Wahl) + fs-managers + Apps
Developer:   Workstation + Forgejo + erweiterte Manager-Tools
```

### API-Standard

```
gRPC (tonic):     primär — intern + extern, type-safe
REST (axum):      zusätzlich — universell, Browser-kompatibel
OpenAPI (utoipa): auto-generiert
NIE direkte Calls über Repo-Grenzen — immer über gRPC/REST
```

### OOP-Grundregeln

```
Design Pattern ZUERST festlegen — Entscheidung vor dem ersten Code
Traits + Structs definieren — erst dann Impl schreiben
Traits statt match-Blöcke, Objekte statt Daten
Immer gegen Interface (Trait) — nie gegen konkrete Impl
Domain-Objekte importieren KEIN fs-render
view.rs ist das einzige Bindeglied zwischen Domain + fs-render
```

### i18n-Grundregeln

```
Jeder user-facing Text braucht FTL — KEINE AUSNAHME
Gilt für: UI, CLI, Fehler, Logs, man-Pages
"Sieht ein Mensch den Text?" → FTL. Interner Code → kein FTL nötig
KEINE Stubs / KEIN "erstmal hardcoded" — immer sofort konkret
Fallback: en (immer eingebaut), weitere Sprachen als Artifacts
keys.rs: FTL-Konstanten — nie Magic Strings im Code
```

### DB-Grundregeln

```
Immer über fs-db Repository<T> + Filter<T>
Konkrete Engine kommt als Artifact (fs-db-engine-sqlite default)
DbEngine-Trait + direktes SeaORM/sqlx nur in Adapter-Repos
```

---

## Offene Konzept-Entscheidungen

Alle O1–O5 aus der vorherigen Version wurden am 2026-04-02 entschieden.
Dokumentiert im Architektur-Überblick oben.

Noch offen:

### O6 — Komponenten-Verzeichnis-Struktur

Wo liegen die TOML-Konfigurations-Dateien der Komponenten?
- Pro Programm: `~/.config/freesynergy/components/{program}/{component}.toml`
- Global: `/etc/freesynergy/components/{component}.toml`
- User überschreibt Global?

```
[ ] Entscheidung: Verzeichnis-Struktur für Komponenten-TOML
[ ] Entscheidung: Priorität User vs. Global vs. Store-Default
```

### O7 — WASM-Komponenten: Sandbox-Grenzen

Welche Capabilities bekommt eine WASM-Komponente?
- Lesen: fs-inventory, fs-session, fs-info (via gRPC)
- Schreiben: nur über Bus-Events (nie direkt)
- UI-Rendering: nur eigene Slot-Area

```
[ ] Entscheidung: WASM-Capability-Set für UI-Komponenten
```

---

---

# Phase 1 — Bootstrap

> Ziel: System kann sich selbst initialisieren und den Store laden.
> Jedes Teilsystem muss standalone funktionieren.
> Standard-Checkliste gilt vollständig.

---

## 1.1 — fs-init: Capability-Detection

```
Design Pattern: Strategy (BootstrapStrategy: GUI | TUI | Headless)

[ ] Design Pattern + Trait-Struktur festlegen (BootstrapStrategy-Trait)
[ ] fs-info einbinden: display_server / terminal / headless ermitteln
[ ] Strategy wählen: GuiBootstrap | TuiBootstrap | HeadlessBootstrap
[ ] Ausgabe vor Engine-Download: NUR println! — keine zusätzlichen Libraries
[ ] i18n: alle println!-Ausgaben als FTL-Keys (en als Fallback hardcoded in Binary)
[ ] cargo fmt + clippy + test grün
```

## 1.2 — fs-init: Install-Wizard

```
Design Pattern: State Machine (WizardStep: Welcome→Capability→Engine→Bundle→Confirm→Progress→Done)

[ ] Design Pattern + Trait-Struktur festlegen (WizardStep-Trait)
[ ] Wizard-Schritte implementieren (State Machine)
[ ] Bundle-Auswahl anzeigen (aus Store/ Katalog)
[ ] Render-Engine-Auswahl: verfügbare Engines aus Store anzeigen, User wählt
[ ] Einzelpaket-Auswahl (falls kein Bundle)
[ ] Install-Target pro Paket: Container | rpm | deb | flatpak | AppImage
[ ] fs-info: OS-Detection → passende Variante vorschlagen
[ ] Adapter immer mitinstallieren (nicht optional)
[ ] Nach Install: Manager startet Konfiguration
[ ] i18n: ALLE User-facing Texte in FTL — keine Ausnahme
[ ] cargo fmt + clippy + test grün
```

## 1.3 — fs-init: Standalone-Test

```
[ ] fs-init ohne jedes andere FS-Service startbar
[ ] Nur fs-info als Dependency (Compile-Time)
[ ] Store-Download funktioniert ohne laufenden fs-store-Container
```

---

---

# Phase 2 — Store

> Ziel: Alles installierbar machen. Store ist Single Source of Truth.
> fs-store läuft als eigenständiger Container.
> Standard-Checkliste gilt vollständig.

---

## 2.1 — Store/ Katalog: Pakete vervollständigen

```
[ ] Alle bestehenden Repos auf einheitliches package.toml-Format migrieren
[ ] Bundles definieren: Minimal | Server | Workstation | Developer
[ ] Bundle "Workstation": render-engine als required-Artifact mit Engine-Wahl
[ ] Für jedes program-Paket: artifact-Einträge für container, rpm, deb, flatpak
[ ] Adapter-Dependencies in package.toml eintragen
[ ] Fork-Pakete: Kanidm, Tuwunel, Stalwart, Zentinel, Zentinel-Plane, Forgejo
[ ] Neue Pakete: Outline, Wiki.js, Bulwark Mail
[ ] Telegram: als reiner Adapter (kein eigener Container — nur API-Integration)
```

## 2.2 — fs-store: Install-Pipeline

```
Design Pattern: Pipeline (Download→Validate→Install→Configure→Register)

[ ] Design Pattern + Trait-Struktur festlegen (InstallStep-Trait)
[ ] InstallRequest: Paket + Target (Container/rpm/deb/flatpak) + Variante
[ ] Download: aus Store-Katalog → URL ermitteln → Artifact holen
[ ] Adapter-Auto-Install: wenn program installiert → zugehöriger Adapter auch
[ ] fs-inventory: nach Install updaten (upsert_resource)
[ ] fs-registry: nach Install registrieren (Register Capability)
[ ] Bus: install::completed publishen
[ ] i18n: ALLE User-facing Texte in FTL
[ ] cargo fmt + clippy + test grün
```

## 2.3 — fs-store: UI + CLI

```
Design Pattern: MVC (StoreController, StoreModel, StoreView via FsView-Trait)

[ ] Design Pattern bestätigen / anpassen
[ ] Store-UI: Paket-Liste, Suche, Detail-View, Install-Wizard
[ ] Install-Wizard: SelectStep → ConfirmStep → ProgressStep → DoneStep
[ ] Engine-Wahl im Wizard: Select/Dropdown aus verfügbaren Engines
[ ] Store-CLI: search / install / remove / list-installed / update
[ ] Bundle-Install: alle Pakete des Bundles auf einmal
[ ] i18n: ALLE User-facing Texte in FTL
[ ] cargo fmt + clippy + test grün
```

## 2.4 — fs-store: Standalone-Test

```
[ ] fs-store ohne fs-desktop startbar (CLI reicht)
[ ] fs-store ohne laufenden fs-node startbar
```

---

---

# Phase 3 — fs-render: Component System

> Ziel: Engine-unabhängiges deklaratives Layout mit WASM-Komponenten.
> Komponenten beschreiben WHAT, Engines rendern HOW.
> Standard-Checkliste gilt vollständig.

---

## 3.1 — LayoutDescriptor (TOML, O6 + O7 müssen entschieden sein)

```
Design Pattern: Interpreter (TOML → LayoutDescriptor → Engine-spezifisches Rendering)

[ ] Design Pattern festlegen
[ ] LayoutDescriptor-Structs (Rust): Shell, Slot, ComponentRef
[ ] TOML-Format definieren: pro Komponente eine eigene .toml-Datei
[ ] Verzeichnis-Struktur (O6 muss entschieden sein)
[ ] fs-config einbinden: TOML laden + validieren
[ ] Hot-Reload: inotify überwacht Komponenten-Verzeichnis
[ ] i18n: alle Labels/Descriptions in FTL
[ ] cargo fmt + clippy + test grün
```

## 3.2 — WASM-Komponenten-System

```
Design Pattern: Plugin (ComponentTrait via fs-plugin-sdk)

[ ] Design Pattern festlegen
[ ] ComponentTrait: render(engine_context, slot_bounds) → LayoutElement
[ ] ComponentId-Registry: Komponente meldet sich beim Start an
[ ] Sandbox-Grenzen definieren (O7 muss entschieden sein)
[ ] fs-plugin-runtime: WASM laden + instanziieren
[ ] Slot-Layout-Algorithmus: top stacken, fill teilt Raum, bottom stacken
[ ] Responsive: Komponente kennt ihre Mindest-/Maxgröße
[ ] i18n: ComponentTrait liefert FTL-Key für seinen Namen/Beschreibung
[ ] cargo fmt + clippy + test grün
```

## 3.3 — Engine-Implementierungen

```
[ ] iced-Impl: LayoutDescriptor → iced-Widgets (fs-gui-engine-iced)
[ ] TUI-Impl: LayoutDescriptor → ratatui-Widgets (neue Impl)
[ ] bevy-Impl: langfristig (fs-gui-engine-bevy)
[ ] Einschränkungen pro Engine dokumentieren (TUI: kein SVG, kein Bild)
[ ] cargo fmt + clippy + test grün (pro Engine-Impl)
```

## 3.4 — Standard-Komponenten (für Desktop)

```
Design Pattern: pro Komponente eigenes Pattern festlegen

[ ] InventoryListComponent: installierte Programme (aus fs-inventory via gRPC)
[ ] PinnedAppsComponent: angepinnte Programme (aus fs-session via gRPC)
[ ] AppSectionsComponent: Programme nach Kategorie
[ ] SystemInfoComponent: CPU/RAM/Disk (aus fs-info via gRPC)
[ ] NotificationBellComponent: Bus-Events
[ ] SearchBarComponent: global search (Phase 5)
[ ] Jede Komponente: eigene .toml-Config + FTL-Keys + Tests
[ ] cargo fmt + clippy + test grün
```

---

---

# Phase 4 — Desktop

> Ziel: Vollständige Desktop-Shell mit konfigurierbarem Layout.
> Läuft auf fs-render + gewählter GUI-Engine.
> Standard-Checkliste gilt vollständig.

---

## 4.1 — Desktop-Shell: Layout-System

```
Design Pattern: Composite (ShellLayout enthält Shells, Shells enthalten Slots)

[ ] Design Pattern festlegen
[ ] ShellLayout: Sidebar | Topbar | Bottombar | Main (alle optional)
[ ] Slot-basierte Konfiguration (aus Phase 3)
[ ] TOML-Config: desktop-layout.toml (hotpluggable)
[ ] Default-Layout: Topbar(brand+breadcrumbs+avatar) + Sidebar(inventory+pinned) + Main
[ ] i18n: ALLE Labels in FTL
[ ] cargo fmt + clippy + test grün
```

## 4.2 — Desktop-Shell: Komponenten einbinden

```
[ ] InventoryList in Sidebar (slot: fill) — aus Phase 3.4
[ ] PinnedApps in Sidebar (slot: bottom) — aus Phase 3.4
[ ] SystemInfo in Bottombar (slot: bottom) — aus Phase 3.4
[ ] NotificationBell in Topbar (slot: top) — aus Phase 3.4
[ ] SearchBar in Topbar (slot: fill) — aus Phase 3.4
[ ] cargo fmt + clippy + test grün
```

## 4.3 — Desktop-Shell: App-Management

```
Design Pattern: Observer (fs-session beobachtet App-Lifecycle via Bus)

[ ] Design Pattern festlegen
[ ] App starten: aus Inventory oder Pinned → FsWindow öffnen
[ ] FsWindow-Trait (fs-render): pro laufende App ein Fenster
[ ] fs-session: app::opened / app::closed via Bus
[ ] Fenster-Liste in Taskbar
[ ] App anpinnen / lösen (persist in fs-session)
[ ] i18n: alle Fenster-Titel + Aktionen in FTL
[ ] cargo fmt + clippy + test grün
```

## 4.4 — Desktop: Settings

```
Design Pattern: Strategy (pro Settings-Seite eigene Strategy-Impl)

[ ] Appearance (Theme-Wahl via fs-theme)
[ ] Language (Sprache global + per App via fs-i18n)
[ ] Desktop-Layout konfigurieren (Slot-Belegung via TOML)
[ ] Service Roles (welcher Service übernimmt welche Funktion)
[ ] Accounts (IAM via Kanidm)
[ ] Shortcuts
[ ] Packages (Store-View)
[ ] i18n: ALLE Texte in FTL
[ ] cargo fmt + clippy + test grün
```

## 4.5 — Desktop: i18n-Abschluss

```
[ ] fs-settings: alle noch hardcodierten Texte in FTL migrieren
[ ] fs-theme-app: FTL-Keys migrieren
[ ] fs-container-app: FTL-Keys migrieren
[ ] fs-gui-workspace: Shell-Texte vollständig in FTL
[ ] cargo fmt + clippy + test grün
```

---

---

# Phase 5 — Ecosystem Services

> Ziel: Kritische Dienste installierbar, konfigurierbar, laufend.
> Jeder Dienst standalone — Integration über Adapter + Bus.
> Standard-Checkliste gilt vollständig.

---

## 5.1 — Kanidm (Auth-Grundlage — Blocker für alle anderen Services)

```
Design Pattern: Adapter (4 Protokoll-Traits: OAuthProvider, ScimProvider, SsoProvider, PamProvider)

[ ] Design Pattern bestätigen (Adapter-Pattern bereits definiert)
[ ] Store-Eintrag: kanidm als fork-Paket (Container + fs-auth-Adapter)
[ ] Konfigurationsassistent nach Install: Admin-Account, Domain, OIDC-Clients
[ ] fs-auth: Kanidm-Impl vollständig (alle 4 Protokoll-Traits)
[ ] Desktop-Login via Kanidm (fs-profile → IAM)
[ ] API-Clients (alle Services): OIDC-Login konfigurierbar
[ ] i18n: ALLE Wizard/Konfig-Texte in FTL
[ ] Standalone-Test: Kanidm ohne fs-desktop startbar
[ ] cargo fmt + clippy + test grün
```

## 5.2 — Zentinel + Zentinel Control Plane (Reverse-Proxy)

```
Design Pattern: Facade (ZentinelManager als Facade über Zentinel API)

[ ] Design Pattern festlegen
[ ] Store-Eintrag: zentinel + zentinel-plane als fork-Pakete
[ ] S3-Integration: Zentinel nutzt opendal für Konfigurations-Storage
[ ] Control Plane: Routen-Konfiguration (Service → Pfad)
[ ] ZentinelManager: konfiguriert Routen nach Service-Install
[ ] fs-registry: neu registrierte Services → Zentinel-Route automatisch
[ ] i18n: ALLE Konfig-Texte in FTL
[ ] Standalone-Test: Zentinel ohne fs-desktop + ohne Kanidm
[ ] cargo fmt + clippy + test grün
```

## 5.3 — Stalwart + Bulwark Mail (E-Mail)

```
Design Pattern: Adapter (MailAdapter: SmtpProvider + ImapProvider)

[ ] Design Pattern festlegen
[ ] Store-Eintrag: stalwart (fork) + bulwark-mail (webmail frontend)
[ ] Konfigurationsassistent nach Install: Domain, MX-Records, TLS
[ ] IAM-Integration: Kanidm via OIDC/LDAP
[ ] Adapter: smtp + imap → fs-registry (Service Role: mail)
[ ] Domain-Konfiguration pro Node
[ ] i18n: ALLE Konfig-Texte in FTL
[ ] Standalone-Test: Stalwart ohne fs-desktop
[ ] cargo fmt + clippy + test grün
```

## 5.4 — Tuwunel (Matrix-Messenger)

```
Design Pattern: Adapter (MatrixAdapter via fs-channel-matrix)

[ ] Design Pattern bestätigen (fs-channel-matrix bereits vorhanden)
[ ] Store-Eintrag: tuwunel (fork) als Container
[ ] fs-channel-matrix: MatrixAdapter vollständig implementieren
[ ] IAM: Matrix-Accounts via Kanidm (OIDC)
[ ] fs-bots: Tuwunel als Bot-Backend (via fs-channel-matrix)
[ ] i18n: ALLE Konfig-Texte in FTL
[ ] Standalone-Test: Tuwunel ohne fs-desktop
[ ] cargo fmt + clippy + test grün
```

## 5.5 — Telegram (externer Kanal — kein eigener Container)

```
Design Pattern: Adapter (TelegramAdapter via fs-channel-telegram)

Hinweis: Telegram läuft EXTERN — FreeSynergy installiert keinen Telegram-Server.
Nur der Adapter (API-Integration + Bot-Support) wird installiert.
Nachgeladen wird er automatisch wenn fs-bots oder fs-channel-telegram installiert wird.

[ ] Design Pattern bestätigen (fs-channel-telegram bereits vorhanden)
[ ] fs-channel-telegram: TelegramAdapter vollständig implementieren
[ ] Store-Eintrag: fs-channel-telegram als adapter-Paket
[ ] fs-bots: Telegram als Bot-Backend (via fs-channel-telegram)
[ ] Konfiguration: Bot-Token via fs-config + Manager
[ ] i18n: ALLE Konfig-Texte in FTL
[ ] cargo fmt + clippy + test grün
```

## 5.6 — Forgejo (Git-Server, Self-Hosted)

```
Design Pattern: Adapter (ForgejoAdapter: GitProvider)

[ ] Design Pattern festlegen
[ ] Store-Eintrag: forgejo als Container-Paket
[ ] IAM: Kanidm OIDC
[ ] S3: Repository-Storage via opendal
[ ] ForgejoAdapter: GitProvider-Trait → fs-registry (Service Role: git)
[ ] i18n: ALLE Konfig-Texte in FTL
[ ] Standalone-Test: Forgejo ohne fs-desktop
[ ] cargo fmt + clippy + test grün
```

## 5.7 — Outline + Wiki.js (Dokumentation / Wiki)

```
Design Pattern: Adapter (WikiAdapter: WikiProvider) — für beide Implementierungen

Hinweis: Beide im Store — guter Test für austauschbare Service-Implementierungen.
Zeigt dass das Adapter-System funktioniert wenn zwei Programme denselben Trait implementieren.

[ ] WikiProvider-Trait definieren (gemeinsame API für Outline + Wiki.js)
[ ] Store-Eintrag: outline als Container-Paket (Standard-Empfehlung)
[ ] Store-Eintrag: wikijs als Container-Paket (Alternative)
[ ] IAM: Kanidm SSO (OIDC) für beide
[ ] S3: Datei-Storage via opendal für beide
[ ] OutlineAdapter + WikiJsAdapter: beide implementieren WikiProvider-Trait
[ ] Service Role: wiki → wählbar zwischen Outline und Wiki.js
[ ] i18n: ALLE Konfig-Texte in FTL
[ ] Standalone-Test: Outline standalone, Wiki.js standalone
[ ] cargo fmt + clippy + test grün
```

---

---

# Phase 6 — Apps & Search

> Alle Apps: UI (FsView-Trait) + CLI + API (gRPC + REST)
> Standalone: jede App läuft ohne den Rest
> Standard-Checkliste gilt vollständig.

---

## 6.1 — Apps: Offene Items

```
fs-db:         CrudRepo → Repository<T> migrieren (low priority)
fs-db:         Filter<T> → SQL übersetzen in Adapter-Repos
fs-bots:       bot-db/src/lib.rs aufteilen (735 Zeilen)
               → conversation.rs + user.rs + state.rs + command_log.rs
fs-bots:       DB nur über fs-db DbEngine-Trait
fs-ai:         Konversationshistorie über fs-db DbEngine-Trait
fs-tasks:      TaskStore über fs-db DbEngine-Trait
fs-lenses:     LensRegistry vollständig (Strategy Pattern)
fs-manager-language: gix-API (pre-0.65) migrieren (bekannter Bug)
matrix-sdk:    PostgreSQL State Store (wartet auf rustc recursion-overflow Fix upstream)
```

## 6.2 — Search

```
Design Pattern: Strategy (SearchStrategy: lokal | registry | host | föderal)

[ ] Design Pattern festlegen
[ ] Search-View (Suchfeld, gruppierte Ergebnisse, Preview)
[ ] Service-Suche (lokal, fs-registry)
[ ] Host-Suche (Bus-aggregiert)
[ ] Föderale Suche
[ ] i18n: ALLE Texte in FTL
[ ] cargo fmt + clippy + test grün
```

---

---

# Phase 7 — Federation & Infrastruktur

> Standard-Checkliste gilt vollständig.

---

## 7.1 — Federation

```
Design Pattern: Observer + Decorator (ActivityPub-Events über Bus)

[ ] Design Pattern festlegen
[ ] Federation-Grundstruktur (ActivityPub)
[ ] Rechte-Kaskade + Audit-Log
[ ] Föderaler Bus
[ ] WireGuard (nach Federation-Grundstruktur)
[ ] Hickory DNS (nach Federation-Grundstruktur)
[ ] i18n: ALLE Texte in FTL
[ ] cargo fmt + clippy + test grün
```

## 7.2 — Infrastruktur

```
[ ] Vaultwarden (Passwort-Manager) — Store-Eintrag + Adapter
[ ] Ntfy / UnifiedPush (Push-Benachrichtigungen) — Store-Eintrag + Adapter
[ ] Element Call + coturn (WebRTC für Matrix-Calls) — Store-Eintrag
[ ] libcosmic: vollständige Integration in fs-gui-engine-iced (G2.8, langfristig)
[ ] i18n: ALLE Texte in FTL
[ ] cargo fmt + clippy + test grün
```

---

---

## Archiviert (Referenz)

```
fs-apps:    archiviert 2026-04-01 (alle Apps in eigene Repos migriert)
fs-builder: archiviert 2026-04-02 (Funktionalität in fs-managers; Pipeline-Pattern erhalten)

Abgeschlossen (nicht mehr in TODO):
fs-bootc, fs-libs (5 Primitives), fs-bus, fs-config, fs-db, fs-i18n, fs-theme,
fs-render (Traits), fs-gui-engine-iced, fs-gui-engine-bevy, fs-web-engine, fs-web-engine-servo,
fs-auth (Traits), fs-registry, fs-inventory, fs-session, fs-info, fs-container,
fs-browser, fs-theme-app, fs-lenses, fs-ai, fs-container-app, fs-tasks, fs-bots,
fs-store (Wizard H9a-H9d), fs-managers, Store/ Katalog (Basis),
fs-documentation, fs-ci, GitHub Actions (alle Repos), Fork-Repos (CI)
```

---

## Reihenfolge

```
1.  O6 + O7: offene Konzept-Entscheidungen (Komponenten-Verzeichnis + WASM-Sandbox)
2.  Phase 5.1: Kanidm (Auth-Grundlage — Blocker für alle anderen Services)
3.  Phase 5.2: Zentinel (Proxy — Grundlage für Service-Routing)
4.  Phase 2: Store (Install-Pipeline + Bundle-Katalog)
5.  Phase 1: Bootstrap (fs-init Wizard + Capability-Detection)
6.  Phase 3: fs-render Component System (Layout-Abstraktion + WASM)
7.  Phase 4: Desktop (Shell + Komponenten)
8.  Phase 5.3–5.7: weitere Ecosystem-Services (Stalwart, Tuwunel, Telegram, Forgejo, Outline/Wiki.js)
9.  Phase 6: Apps & Search
10. Phase 7: Federation & Infrastruktur
```
