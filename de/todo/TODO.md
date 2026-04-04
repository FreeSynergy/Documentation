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

5b. Bibliotheken aktuell halten (bei jeder Arbeit am Repo):
   → cargo upgrade --incompatible (cargo-edit) oder cargo update
   → Breaking Changes prüfen + anpassen
   → Build + Tests danach wieder grün sicherstellen
   → Dependency-Updates als separater Commit vor dem Feature-Commit

6. Dokumentation — PFLICHT, nicht optional:
   → Doku-Seite in fs-documentation/de/ anlegen oder aktualisieren
   → Neue Konzepte: konzepte/{modul}.md
   → Neue Programme: programme/{modul}.md
   → Neue Technik/Architektur: technik/{modul}.md oder architektur/{modul}.md
   → INDEX.md aktualisieren (Link zur neuen/geänderten Seite)
   → fs-documentation committen + pushen
   → Erst danach: das eigentliche Modul-Repo committen + pushen
   → Gilt für JEDES Modul — keine Ausnahme
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

gRPC-First-Regel (entschieden 2026-04-03):
  KEIN raw JSON/serde zwischen FS-Services — immer gRPC (tonic + protobuf)
  REST nur für externe Clients (Browser, 3rd-party Tools)
  serde/JSON NUR wenn keine Alternative existiert (z.B. Kanidm-REST-API als 3rd-party Fork)
  Library-Crates (heute noch embedded) → sobald als eigener Container → gRPC-Interface drüber
  Ausnahmen IMMER kommentieren: // gRPC not possible: external API (Kanidm/Forgejo/...)
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

## Konzept-Entscheidungen

Alle O1–O7 wurden entschieden (O1–O5: 2026-04-02, O6–O7: 2026-04-02).

### O6 — Komponenten + Paket-Deklaration (entschieden 2026-04-02)

```
Komponenten werden im Paket selbst deklariert — in package.toml des jeweiligen Pakets.
Jedes Paket kennt seine Speicher-Pfade:
  storage.user   = "~/.local/share/freesynergy/{paket}"   (privat, pro User)
  storage.global = "/var/lib/freesynergy/{paket}"         (global, systemweit)
  storage.config = "~/.config/freesynergy/{paket}"        (Konfig, pro User)
  storage.cache  = "~/.cache/freesynergy/{paket}"         (Cache, pro User)

Komponenten-Deklaration in package.toml:
  [[component]]
  id      = "inventory-list"
  slot    = "fill"           ← bevorzugter Slot
  wasm    = "components/inventory_list.wasm"
  config  = "components/inventory_list.toml"   ← Default-Einstellungen

Paket-Dokumentation enthält:
  - Eigener API-Tab: alle gRPC-Endpoints + REST-Endpoints
  - Komponenten-Tab: alle Komponenten mit Default-Einstellungen
  - Storage-Tab: User/Global/Config/Cache Pfade
```

### O7 — WASM-Komponenten: Sandbox-Grenzen (entschieden 2026-04-02)

```
Lesen:    fs-inventory, fs-session, fs-info — NUR via gRPC (nie direkt)
Schreiben: NUR via Bus-Events (nie direkt in DB oder Dateisystem)
UI:       NUR eigene Slot-Area (kein Zugriff auf andere Slots)
Netzwerk: kein direkter Netzwerkzugriff (alles über FS-Services)
Falls andere Grenzen nötig werden: in der Session besprechen.
```

---

---

# Phase 1 — Bootstrap

> Ziel: System kann sich selbst initialisieren und den Store laden.
> Jedes Teilsystem muss standalone funktionieren.
> Standard-Checkliste gilt vollständig.

---

## 1.1 — fs-init: Capability-Detection ✅

```
Design Pattern: Strategy (BootstrapStrategy: GUI | TUI | Headless)
Umgesetzt 2026-04-03

fs-info: WaylandDisplay, X11Display, Terminal Features hinzugefügt
BootstrapCapability: erkennt display / terminal / container / mode
GuiBootstrap | TuiBootstrap | HeadlessBootstrap implementiert
Ausgabe: nur println! — keine zusätzlichen Libraries
i18n: alle Texte als FTL-Keys in keys.rs (en Fallback hardcoded)
cargo fmt + clippy + test grün ✅
```

## 1.2 — fs-init: Install-Wizard ✅

```
Design Pattern: State Machine (WizardStep-Trait, 8 Schritte)
Umgesetzt 2026-04-03, überarbeitet 2026-04-03

Welcome → Capability → StoreLoad → Engine → Bundle → Confirm → Progress → Done

StoreLoad: klont Store-Katalog (non-fatal bei Fehler, Fallback Built-in Defaults)
Bundle-Auswahl: dynamisch aus lokalem Store-Katalog (catalog_reader.rs)
  BundleChoice/EngineChoice: String-Felder (kein &'static str mehr)
Render-Engine-Auswahl: iced | bevy | tui | none
Install-Target: Container | RPM | DEB | AppImage (OS-Detection)
Progress: echte Install-Pipeline (fs-store Pipeline, je Komponente)
  Adapter werden automatisch mitinstalliert (AdapterInstallStep)
Done: fs-manager auto-launch + Hinweise auf nächste Schritte
gix: 0.80 (0.81 noch nicht vollständig auf crates.io)
i18n: ALLE User-facing Texte als FTL-Keys ✅
cargo fmt + clippy + test grün ✅
```

## 1.3 — fs-init: Standalone-Test ✅

```
fs-init ohne andere FS-Services startbar ✅
Nur fs-info als Compile-Time-Dependency ✅
Store-Clone ohne laufenden fs-store-Container ✅ (gix, kein System-git)
```

---

---

# Phase 2 — Store ✅ 2026-04-03

> Abgeschlossen. Store-Katalog vollständig, Install-Pipeline live, CLI mit install/remove/update.


---

---

# Phase 3 — fs-render: Component System ✅ 2026-04-03

> Abgeschlossen. Deklaratives Layout mit WASM-Sandbox + 3 Engine-Impls + 5 Standard-Komponenten.

## 3.1 ✅ LayoutDescriptor (TOML)

```
Design Pattern: Interpreter — TOML → LayoutDescriptor → Engine-spezifisches Rendering
fs-render: layout.rs (LayoutDescriptor, ShellConfig, SlotConfig, ComponentRef, StoragePaths)
Layered load: Store-Default < Global < User — fehlende Schichten übersprungen
Hot-Reload: HotReloadWatcher (notify/inotify) → HotReloadEvent (LayoutChanged/ComponentChanged)
i18n: fs-i18n/locales/{en,de}/render.ftl — alle User-facing Texte als FTL-Keys
cargo fmt + clippy + test grün (77 Tests) ✅
Dokumentation: konzepte/component-system.md ✅
```

## 3.2 ✅ WASM-Komponenten-System

```
Design Pattern: Plugin — ComponentTrait + ComponentRegistry
ComponentTrait: component_id, name_key, description_key, slot_preference, min_width/height, render
LayoutElement-Baum: Text, Button, Icon, Row, Column, List, Separator, Badge, Spinner, Spacer
Sandbox-Grenzen O7: gRPC read-only, Bus-Events write-only, eigene Slot-Area, kein Netzwerk
fs-plugin-runtime: wasmtime + WASI P1 Sandbox (PluginSandbox::minimal/allow_read/allow_write)
Dokumentation: konzepte/wasm-komponenten.md ✅
```

## 3.3 ✅ Engine-Implementierungen

```
iced (fs-gui-engine-iced):
  iced_aw 0.11 — Spinner (Loading-Animation), Badge (Pills), Card (Panel)
  IcedLayoutInterpreter: LayoutDescriptor → Element<'static, LayoutMessage>
  cargo fmt + clippy + test grün (28 Tests) ✅

TUI (fs-gui-engine-tui) — NEU:
  ratatui 0.30 + tachyonfx 0.25 (fx::fade_from — Fade-In Animationen)
  TuiLayoutInterpreter → TuiLayoutOutput (Lines + Animations)
  shell_rects() / center_rects() Layout-Helfer
  cargo fmt + clippy + test grün ✅

Bevy: langfristig (fs-gui-engine-bevy) — bevy_tweening geplant
Einschränkungen TUI dokumentiert (kein SVG/Bild)
```

## 3.4 ✅ Standard-Komponenten (für Desktop)

```
5 Komponenten in fs-render/src/components/:
  InventoryListComponent  — Sidebar/fill — fs-inventory gRPC
  PinnedAppsComponent     — Sidebar/bottom — fs-session gRPC
  AppSectionsComponent    — Main/fill — fs-inventory Kategorien
  SystemInfoComponent     — Bottombar/bottom — fs-info gRPC
  NotificationBellComponent — Topbar/top — fs-bus Notification
register_standard_components() — registriert alle 5 beim Start
SearchBarComponent → Phase 6 (Search)
cargo fmt + clippy + test grün ✅
```

---

---

# Phase 4 — Desktop ✅

> Umgesetzt 2026-04-04. Vollständige Desktop-Shell mit konfigurierbarem Layout.

---

## 4.1 ✅ Desktop-Shell: Layout-System

Composite Pattern: `ShellLayout` → `ShellSection` → `SlotEntry`.
TOML-Config: `~/.config/freesynergy/desktop/desktop-layout.toml`.
Default: Topbar(`notification-bell`) + Sidebar(`inventory-list`/`pinned-apps`) + Main + Bottombar(hidden).

## 4.2 ✅ Desktop-Shell: Komponenten einbinden

`SharedComponentRegistry` in `DesktopShell`; `register_standard_components()` registriert Phase-3-Komponenten.

## 4.3 ✅ Desktop-Shell: App-Management

Observer Pattern: `AppLifecycleBus` mit `SessionLifecycleObserver`. Events: Opened/Closed/Pinned/Unpinned.
Pin/Unpin via `PackageRegistry::set_pinned`. Sidebar zeigt Pin-Button je App.

## 4.4 ✅ Desktop: Settings — Layout-Seite

Strategy Pattern: `LayoutSectionStrategy` → `TopbarStrategy`, `SidebarStrategy`, `BottombarStrategy`.
Settings-Seite `Layout` in `fs-settings` mit TOML-Proxy (keine Zirkelabhängigkeit zu `fs-gui-workspace`).

## 4.5 ✅ Desktop: i18n-Abschluss

`fs-i18n/locales/{en,de}/desktop.ftl` mit allen Shell- und Profil-Keys.
`SnippetPlugin` + TOML-Snippets aus `fs-gui-workspace` entfernt → `init_with_builtins()`.
Alle Keys auf Bindestriche migriert (z.B. `shell-menu-about`).

---

## Offen aus Phase 4 (Folge-Phasen)

```
[ ] Appearance-Settings (Theme-Wahl via fs-theme)
[ ] Language-Settings (Sprache global + per App)
[ ] Service Roles Settings
[ ] Accounts-Settings (IAM via Kanidm)
[ ] Shortcuts-Settings
[ ] Packages-Settings (Store-View)
[ ] SearchBar in Topbar (slot: fill) — Phase 6
[ ] Hot-Reload Layout via inotify (Phase 3 HotReloadWatcher)
[ ] IcedLayoutInterpreter vollständig in Shell-View einbinden
[ ] OIDC-Login-Flow in fs-settings konfigurierbar
```

---

## Phase 4B — Desktop: Visual & UX ✅ 2026-04-04

```
Design Pattern: State Machine (SidebarState) + Strategy (HelpSource) + Observer (MouseProximityObserver)

4B.1 SVG-Fix: ✅
  Binary-Name: [[bin]] name = "fs-desktop" (war bereits korrekt)
  IcedLayoutInterpreter: Icon → render_icon() mit SVG-Ladeversuch (3 Pfade) + Emoji-Fallback
  cargo fmt + clippy + test grün ✅

4B.2 Sidebar State Machine: ✅
  SidebarState: Collapsed (48px) | Expanding | Open — sidebar_state.rs
  SidebarMode: Auto | Pinned — Pin-Button in jeder Sidebar
  MouseProximityObserver: Proximity-Check (16px Schwellwert) — transitions CursorMoved

4B.3 Linke Sidebar (Taskbar): ✅
  Collapsed: nur Icon-Strip (⊞, App-Icons, ⚙)
  Expanded: Icons + Labels + Pin-Buttons + Section-Labels
  Settings ⚙ immer sichtbar am unteren Ende — nie verschiebbar
  Trennlinie installed / pinned ✅

4B.4 Rechte Sidebar (Help + AI): ✅
  HelpSource-Trait: LocalHelpTopicSource | AiHelpSource | NoHelpSource
  ActiveWindowObserver: maps app_id → HelpContext-Key
  CapabilityCheck: FS_AI_ENDPOINT env var / Flag-Datei → AiInputBar einblenden
  AiInputBar: TextInput + Send (submit on Enter oder Button)

4B.5 Sidebar-Position: ✅
  ShellSection.position = SidebarSide::Left | Right (TOML-serialisierbar)
  sidebars_on_side(side) Methode auf ShellLayout
  Default: LeftSidebar (Taskbar) + RightSidebar (Help)

i18n: desktop.ftl (en + de) — 15 neue Keys ✅
cargo fmt + clippy + test grün (27 Tests in fs-gui-workspace, 28 in fs-gui-engine-iced) ✅
Dokumentation: programme/fs-desktop.md ✅

Noch offen (Folge-Phasen):
[ ] Hot-Reload Layout via inotify (Phase 3 HotReloadWatcher einbinden)
[ ] IcedLayoutInterpreter vollständig in Shell-View einbinden
[ ] OIDC-Login-Flow in fs-settings konfigurierbar
[ ] AiHelpSource: echte gRPC-Anbindung an fs-ai (Phase 6)
[ ] SearchBar in Topbar (slot: fill) — Phase 6
[ ] konzepte/help-sidebar.md anlegen (Phase 6)
```

---

# Phase 5 — Ecosystem Services

> Ziel: Kritische Dienste installierbar, konfigurierbar, laufend.
> Jeder Dienst standalone — Integration über Adapter + Bus.
> Standard-Checkliste gilt vollständig.

---

## 5.1 — Kanidm ✅ 2026-04-03

```
Design Pattern: Adapter (4 Protokoll-Traits) + State Machine (KanidmSetupWizard)

fs-auth: KanidmBackend (OAuthProvider, ScimProvider, SsoProvider, PamProvider) ✅
Store-Eintrag: kanidm als fork-Container + fs-auth-Adapter ✅
KanidmSetupWizard: Domain → Admin → OidcClients → Confirm → Done ✅
  → fs-managers/auth (fs-manager-auth)
PamIdentity: session_token-Feld (Bearer direkt aus PAM Step 3) ✅
OidcClientManager: Post-Wizard OIDC-Client-Verwaltung + sync_to_kanidm ✅
Desktop-Login: fs-profile Login-Screen + AuthState (LoggedOut/Authenticating/LoggedIn) ✅
  → FS_AUTH_URL env var → Login-Screen; leer → Offline-Modus
Integration-Tests: fs-auth/tests/kanidm_integration.rs (6 Tests, #[ignore]) ✅
i18n: fs-i18n/locales/{en,de}/auth-setup.ftl ✅ (+ auth-manager-* Keys)
cargo fmt + clippy + test grün ✅

Noch offen (Phase 4/6):
[ ] Standalone-Test mit laufendem Kanidm-Container (env vars gesetzt)
[ ] OIDC-Login-Flow in fs-settings konfigurierbar (Desktop-Settings-Seite)
```

## 5.2 — Zentinel ✅ 2026-04-03

```
Design Pattern: Facade (ZentinelManager) + Observer (ZentinelBusHandler)

Store-Einträge: zentinel + zentinel-plane als fork-Pakete ✅
ZentinelManager: Facade über Zentinel Control Plane API ✅
RouteConfig + RouteTable + ZentinelBusHandler ✅
Auto-Routing: registry::service::registered → Zentinel-Route ✅
Capability → Pfad-Mapping (/auth, /git, /mail, /wiki, /chat, /storage) ✅
i18n: fs-i18n/locales/{en,de}/zentinel.ftl ✅
cargo fmt + clippy + test grün (22 Tests) ✅

Noch offen:
[ ] S3-Integration: Zentinel nutzt opendal für Konfigurations-Storage
[ ] Standalone-Test mit laufendem Zentinel-Container
```

## 5.3 — Stalwart + Bulwark Mail (E-Mail) ✅ 2026-04-04

```
Design Pattern: Adapter (3 Protokoll-Traits) + State Machine (StalwartSetupWizard)

fs-mail: SmtpProvider + ImapProvider + JmapProvider Traits ✅
  StalwartBackend: implements alle 3 Traits + MailBackend ✅
  MailCapabilities: mail.smtp / mail.imap / mail.jmap ✅
  MailEvent: mail.received / mail.sent / mail.adapter.* ✅
  Features: stalwart (reqwest), bus (fs-bus) ✅
Store: stalwart catalog.toml — [adapter] + JMAP/web-admin routes ✅
Store: bulwark catalog.toml — bereits vorhanden ✅
StalwartSetupWizard: Domain → TlsCerts → OidcIntegration → Confirm → Done ✅
  ACME (Let's Encrypt) oder manuelle Cert/Key-Pfade ✅
  Kanidm OIDC (skippable für lokale Accounts) ✅
i18n: fs-i18n/locales/{en,de}/mail.ftl — alle User-facing Texte ✅
DNS-Hinweise: MX, SPF, DKIM, DMARC im Manager-UI ✅
18 Tests (fs-mail) + 15 Tests (fs-manager-mail) ✅
cargo fmt + clippy + test grün ✅
Dokumentation: programme/fs-mail.md ✅

Noch offen:
[ ] Standalone-Test mit laufendem Stalwart-Container (env vars gesetzt)
[ ] JMAP-Login-Flow in fs-settings konfigurierbar
```

## 5.4 — Tuwunel (Matrix-Messenger) ✅ 2026-04-04

```
Design Pattern: Adapter (MatrixBotAdapter via fs-channel matrix-bot feature) + State Machine (TuwunelSetupWizard)

fs-channel: matrix-bot Feature (CS-API via reqwest, kein matrix-sdk) ✅
  MatrixBotAdapter: BotChannel impl (polling, send, send_dm, send_menu) ✅
  ChannelRegistry::build_bot Matrix-Arm ✅
fs-bots: matrix-bot Feature aktiviert (neben telegram) ✅
Store: tuwunel catalog.toml (setup + iam_required + manager) ✅
fs-channel-matrix: keys.rs um ADAPTER_REGISTERED/DEREGISTERED erweitert ✅
  5 Tests grün ✅
fs-manager-matrix: TuwunelSetupWizard (State Machine, 6 Schritte) ✅
  ServerName → TlsCerts → OidcIntegration → Federation → Confirm → Done
  Kanidm OIDC Pflicht: skip nur für Offline-Tests erlaubt ✅
  FsView + ManagerLayout (view.rs) ✅
i18n: fs-i18n/locales/{en,de}/matrix.ftl (alle Wizard-Texte + IAM-Hinweise) ✅
cargo fmt + clippy + test grün (fs-channel, fs-bots, fs-manager-matrix, fs-channel-matrix) ✅
Dokumentation: programme/fs-matrix.md + INDEX.md ✅

Noch offen:
[ ] Standalone-Test mit laufendem Tuwunel-Container (env vars gesetzt)
[ ] matrix-sdk live Feature: warten auf upstream rustc ≥1.94 Fix (matrix-sdk 0.16)
```

## 5.5 — Telegram (externer Kanal — kein eigener Container) ✅ 2026-04-04

```
Design Pattern: Adapter (TelegramAdapter via fs-channel-telegram)

Umgesetzt 2026-04-04

BotChannel-Trait für TelegramAdapter implementiert (polling, send, menus) ✅
  receive_updates: getUpdates polling mit Offset-Tracking ✅
  send / send_formatted / send_dm / send_menu: inline keyboards ✅
ChannelRegistry::build_bot() für Telegram verdrahtet ✅
TelegramChannelConfig über fs-config (TelegramConfigStore) ✅
  bot_token_ref als Geheimreferenz (env:VAR / file:PATH) — nie Klartext ✅
fs-manager-telegram: Setup-Wizard (State Machine, 3 Schritte) ✅
  fs-telegram setup / status / show / set-token / set-chats ✅
i18n: fs-i18n/locales/{en,de}/channel-telegram.ftl vollständig ✅
Store-Catalog: [storage] + [api.grpc] + [api.rest] + [notes] ✅
cargo fmt + clippy + test grün ✅

Noch offen:
[ ] Standalone-Test mit echtem Bot-Token (env vars gesetzt)
[ ] gRPC subscribe: Streaming-Impl (derzeit deferred → REST long-poll)
```

## 5.X — fs-pod-forge: Container YAML Configurator

```
Design Pattern: Strategy (PodConfigurator-Trait) + Template Method (BasePodConfigurator)
               Builder (PodManifestBuilder für pod.yml Konstruktion)

Konzept:
  Jedes Container-Paket bringt seine eigene PodConfigurator-Impl mit (als feature im package).
  Manager ruft PodConfigurator::generate() auf → valides pod.yml.
  export_yaml() gibt standalone-fähiges YAML aus (ohne laufende FS-Services).
  validate() prüft semantische Korrektheit (Ports, Volumes, Env-Vars).

Repos:
  fs-pod-forge  — Trait-Definition + BasePodConfigurator + Builder
  Pakete (kanidm, stalwart, etc.) implementieren PodConfigurator jeweils selbst

[ ] Design Pattern festlegen + dokumentieren
[ ] PodConfigurator-Trait definieren:
    → generate(config: &PodConfig) -> PodManifest
    → validate(manifest: &PodManifest) -> ValidationResult
    → export_yaml(manifest: &PodManifest) -> String  (standalone-fähig)
    → diff(old: &PodManifest, new: &PodManifest) -> ManifestDiff
[ ] PodManifest-Typen: Container, Volume, Port, EnvVar, Secret, NetworkPolicy
[ ] BasePodConfigurator: gemeinsame Logik (Port-Clash-Check, Secret-Referenz-Auflösung)
[ ] PodManifestBuilder: typsicherer Builder für pod.yml (kein raw YAML String)
[ ] Manager-Integration: ManagerAction::UpdatePodConfig → PodConfigurator::apply()
    → podman play kube --replace <generated.yml>
[ ] SystemdIntegration: generate_systemd_unit() für Container-Service
    → systemctl enable + start via fs-managers
[ ] CLI: fs-pod generate <paket> | validate <file> | diff <old> <new>
[ ] Erste Implementierungen: KanidmPodConfigurator, StalwartPodConfigurator
[ ] i18n: pod-forge.ftl (alle User-facing Texte)
[ ] cargo fmt + clippy + test grün
[ ] Dokumentation: technik/fs-pod-forge.md
```

## 5.X+1 — fs-app-forge: App Config File Configurator

```
Design Pattern: Strategy (AppConfigurator-Trait) + Schema-driven UI (ConfigSchema → Form)
               Visitor (ConfigValidator besucht Schema-Knoten)

Konzept:
  Jedes Paket bringt seine eigene AppConfigurator-Impl mit.
  Schema-Prinzip: AppConfigurator::schema() liefert strukturiertes Schema →
    Manager generiert automatisch das Konfigurations-Formular (kein hardcodiertes UI).
  apply_changes() schreibt sicher in die Config-Datei (Backup + atomic write).
  Unterschiedliche Formate: TOML, YAML, HOCON, .env — per Feature-Flag.

Repos:
  fs-app-forge  — Trait-Definition + ConfigSchema + FormGenerator
  Pakete implementieren AppConfigurator jeweils selbst

[ ] Design Pattern festlegen + dokumentieren
[ ] AppConfigurator-Trait definieren:
    → schema() -> ConfigSchema
    → read() -> ConfigValues
    → validate(values: &ConfigValues) -> ValidationResult
    → apply(changes: ConfigChanges) -> Result<()>  (atomic write + backup)
    → export() -> String  (lesbare Config-Datei für Standalone-Nutzung)
[ ] ConfigSchema-Typen: Section, Field (String/Int/Bool/Enum/Secret/Path/List)
    → Field hat: key, label_key (FTL), description_key (FTL), default, required, validator
[ ] ConfigFormGenerator: ConfigSchema → LayoutDescriptor (automatisches UI)
    → Kein hardcodiertes Formular — alles aus Schema generiert
[ ] Format-Adapter: TomlConfigAdapter, YamlConfigAdapter, EnvConfigAdapter
[ ] SecretField-Handling: Werte nie im Klartext — Referenz auf env:VAR / file:PATH
[ ] Manager-Integration: ManagerAction::EditConfig → AppConfigurator::apply()
[ ] CLI: fs-forge show <paket> | edit <paket> <key>=<val> | export <paket>
[ ] Erste Implementierungen: KanidmAppConfigurator, StalwartAppConfigurator
[ ] i18n: app-forge.ftl (alle User-facing Texte)
[ ] cargo fmt + clippy + test grün
[ ] Dokumentation: technik/fs-app-forge.md
```

## 5.X+2 — Manager-Upgrade: Service Controller + Category Manager ✅

Erledigt 2026-04-04:
- ServiceController-Trait + SystemdServiceController + ContainerServiceController (podman pod)
- CategoryManager-Trait + update_available() Default-Methode
- KanidmIamController, StalwartMailController, TuwunelMessengerController, ZentinelProxyController
- Services-Tab in allen Manager-Views (auth/mail/matrix/zentinel)
- Doku: konzepte/manager-service-controller.md aktualisiert
- cargo fmt + clippy + test grün

Offen (G2):
[ ] Role-Switching: fs-registry Capability-Eintrag umschreiben wenn set_active() aufgerufen
[ ] Update-Check: update_available() mit echtem Store-Lookup implementieren

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

## 5.7 — Outline + Wiki.js (Dokumentation / Wiki) ✅ 2026-04-04

```
Design Pattern: Strategy (WikiProvider — OutlineAdapter / WikiJsAdapter)
              + State Machine (WikiSetupWizard)
              + Composite (WikiCategoryController — beide Impl.)

✅ WikiProvider-Trait definieren (gemeinsame API für Outline + Wiki.js)
✅ Store-Eintrag: outline als Container-Paket (Standard-Empfehlung)
✅ Store-Eintrag: wikijs als Container-Paket (Alternative)
✅ IAM: Kanidm SSO (OIDC) für beide (pod.yml + OidcConfig im Wizard)
✅ S3: Datei-Storage via opendal für beide (S3Config im Wizard, optional)
✅ OutlineAdapter + WikiJsAdapter: beide implementieren WikiProvider-Trait
✅ Service Role: wiki → wählbar zwischen Outline und Wiki.js (WikiCategoryController)
✅ i18n: ALLE Konfig-Texte in FTL (en + de in fs-i18n)
✅ Standalone-Test: 23 Unit-Tests grün (provider, wizard, service_controller)
✅ cargo fmt + clippy + test grün
```

---

---

# Phase 5B — Store: Catalog-Vollständigkeit + Browsability

> Ziel: Alle Programme sind im Store sichtbar + beschreibbar — auch vor der Installation.
> Store ist der erste Eindruck des Systems. Jedes Paket muss vollständig sein.
> Standard-Checkliste gilt vollständig.

## 5B.1 — Pflicht-Felder für jeden Store-Eintrag

```
Design Pattern: Template Method (PackageCatalogValidator prüft Vollständigkeit)

Pflichtfelder pro catalog.toml:
  [package]
    name, version, description (kurz, 1 Satz), license
  [display]
    title        — Anzeigename (z.B. "Kanidm — Identity & Access Management")
    summary      — 2-3 Sätze was das Programm macht
    description  — ausführlich (Markdown) — für Store-Detailseite
    icon         — Pfad zum SVG-Icon
    screenshots  — Liste von Screenshot-Pfaden (mindestens 1 wenn GUI)
    tags         — Kategorien (z.B. ["iam", "auth", "security"])
    homepage     — Upstream-URL
    changelog    — Verweis auf CHANGELOG oder Release-Notes

[ ] PackageCatalogValidator: prüft alle Pflichtfelder bei catalog.toml-Parse
    → Warnung wenn Felder fehlen (kein Hard-Fail bei install, aber im Store sichtbar)
[ ] Store-UI: "Incomplete" Badge wenn Felder fehlen
[ ] Alle bestehenden catalog.toml ergänzen:
    → kanidm, stalwart, tuwunel, zentinel, telegram, outline, wikijs
    → fs-desktop, fs-init, fs-store, fs-auth, fs-managers
    → Render-Engines, DB-Engines (als Artifacts)
[ ] Screenshots-System: Store zeigt Bilder im Detail-Panel
    → iced: iced::widget::image (PNG) — SVG für Icons, PNG für Screenshots
[ ] i18n: store-package-incomplete, store-package-no-description in store.ftl
[ ] cargo fmt + clippy + test grün
[ ] Dokumentation: technik/store-catalog-spec.md
```

## 5B.2 — Store-Browsability: Kategorien + Suche vor Installation

```
Design Pattern: Strategy (BrowseStrategy: Installed | Available | All)
               Filter (CategoryFilter, TagFilter, SearchFilter)

[ ] BrowseMode: User kann Store öffnen ohne etwas zu installieren
    → "Verfügbar" Tab: alle Pakete aus Store/ Katalog
    → "Installiert" Tab: aus fs-inventory
    → "Updates" Tab: Diff zwischen installiert + Store-Version
[ ] Kategorie-Navigation: Tags aus catalog.toml → Seitenleiste im Store-UI
    → IAM / Mail / Chat / Git / Wiki / Desktop / Tools / Engines / Themes / Bots
[ ] Vorschau-Panel: Beschreibung + Screenshots + Version + License — ohne Install
[ ] "Installieren"-Button nur wenn fs-init / fs-store Service läuft
[ ] i18n: store-browse-available, store-browse-installed, store-category-* in store.ftl
[ ] cargo fmt + clippy + test grün
```

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

Phase 2 ✅ 2026-04-03 (Restpunkte ✅ 2026-04-03):
Store/ Katalog: [storage]+[api]+[adapter]+[install_targets] für alle Kern-Pakete,
Developer-Bundle, Workstation render-engine-Wahl, Wiki.js + Bulwark Mail.
Outline + Telegram Adapter: [storage]+[api]+[install_targets] nachgetragen.
fs-store: Install-Pipeline (7 Steps), BundleInstallStep (Sub-Pipeline),
EngineSelectStep (Wizard), StoragePaths + ApiEndpoint, Storage/API-Tabs in
detail_panel, CLI install/remove/update. cargo fmt + clippy + tests grün.
```

---

## Reihenfolge

```
1.  O6 + O7: offene Konzept-Entscheidungen ✅
2.  Phase 5.1: Kanidm ✅ | Phase 5.2: Zentinel ✅
3.  Phase 2: Store ✅ 2026-04-03
4.  Phase 1: Bootstrap ✅ | Phase 3: fs-render ✅ | Phase 4: Desktop ✅
5.  Phase 5.3–5.5: Stalwart ✅, Tuwunel ✅, Telegram ✅

--- Aktuell offen ---

6.  Phase 4B: Desktop Visual + UX (Binary-Fix, Sidebar, SVG, Help, AI)
7.  Phase 5B: Store Catalog-Vollständigkeit + Browsability
8.  Phase 5.X:  fs-pod-forge (Container YAML Configurator)
9.  Phase 5.X+1: fs-app-forge (App Config File Configurator)
10. Phase 5.X+2: Manager-Upgrade (Service Controller + Category Manager)
11. Phase 5.6: Forgejo | Phase 5.7: Outline + Wiki.js ✅ 2026-04-04
12. Phase 6: Apps & Search
13. Phase 7: Federation & Infrastruktur
```
