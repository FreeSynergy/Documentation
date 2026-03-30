# FreeSynergy — Build Plan

[← Zurück zum Index](../INDEX.md)

---

## Regeln für Claude-Sessions

- TODO vollständig lesen zu Beginn jeder Session
- Erledigte Items löschen — kein [✓] behalten, spart Token
- Vor jeder Entscheidung fragen — nie eigenmächtig handeln
- Repo-Arbeit immer: erst lesen → dann planen → dann umsetzen

---

## Status-Markierungen

```
[ ]  Muss noch gemacht werden
[o]  Wird gerade bearbeitet
[~]  Muss noch getestet werden
[✓]  Erledigt → sofort löschen
```

---

## Architektur-Überblick (Stand 2026-03-29)

### 3-Stufen-Installation

```
fs-init
  └── lädt Store
        ├── Bundles  (definiert in Store/ Katalog)
        │     z.B. "Workstation" = fs-desktop + fs-gui-engine-iced + fs-managers + …
        │     z.B. "Server"      = fs-node + fs-auth + fs-s3 + …
        └── Einzelpakete (frei wählbar)
              └── Artifacts (optionale Teile eines Pakets, nachladen)
                    z.B. fs-i18n-de (Sprache global oder per-Paket)
                    z.B. fs-theme-dark (Theme)
```

### 4 Package-Typen im Store

```
bundle   — Gruppe von Paketen (z.B. Workstation, Server)
           Definiert in Store/ Katalog als bundle.toml
program  — Laufender Dienst als Container (z.B. fs-store, fs-desktop, fs-node)
adapter  — Implementiert einen Trait, registriert Capability in fs-registry
           (z.B. fs-db-engine-postgres, fs-llm-mistral)
artifact — Reine Daten, kein laufender Prozess
           (z.B. fs-i18n-de, fs-theme-dark, Icon-Sets)
```

### Artifact: Global vs. Per-Paket

```
Global installieren:
  → gilt für ALLE installierten Pakete
  z.B. Sprache "de" global  → alle Programme auf Deutsch
  z.B. PostgresEngine global → alle DB-Nutzer nutzen Postgres

Per-Paket installieren:
  → gilt nur für dieses eine Paket
  z.B. Sprache "de" nur für fs-store → nur Store auf Deutsch

Gilt für: Sprachen, Themes, DB-Engines, LLM-Adapter, Render-Engines, Bots, …
```

### Namenskonvention Adapter-Repos

```
{domain}-engine-{impl}  wenn es eine austauschbare Ausführungs-Engine ist:
  fs-db-engine-sqlite
  fs-db-engine-postgres
  fs-gui-engine-iced     (bereits so benannt)
  fs-gui-engine-bevy     (bereits so benannt)
  fs-web-engine-servo    (bereits so benannt)

{domain}-{impl}  wenn es ein Adapter zu einem externen Service ist:
  fs-llm-mistral
  fs-llm-openai
  fs-channel-matrix
  fs-channel-telegram
```

### Repo = Container (Ausnahmen)

```
Hat Containerfile:    alle program + adapter Repos
Kein Containerfile:   fs-libs (Compile-Time), Store/ (Metadaten-Katalog)
Sonderfall fs-i18n:   ist ein Service (i18n-Lookup via gRPC) → hat Containerfile
```

### API-Standard (alle Services)

```
gRPC   (tonic):  primär — intern + extern, type-safe
REST   (axum):   zusätzlich — universell, Browser-kompatibel
OpenAPI (utoipa): auto-generiert aus axum-Handlern
NIE direkte Calls über Repo-Grenzen — immer über gRPC/REST
```

### 3 UI-Targets (nur sinnvolle implementieren)

```
UI:  FsView-Trait aus fs-render — Engine entscheidet Rendering
CLI: clap — alle Operationen auch ohne GUI
API: gRPC + REST — Basis, immer wenn Service läuft
```

### OOP-Grundregeln

```
Design Pattern ZUERST festlegen
Traits + Structs definieren — erst dann Impl schreiben
Traits statt match-Blöcke, Objekte statt Daten
Immer gegen Interface (Trait) — nie gegen konkrete Impl
Domain-Objekte importieren KEIN fs-render
View-Impl (view.rs) ist das einzige Bindeglied zwischen Domain + fs-render
```

### i18n-Grundregeln

```
Jeder user-facing Text braucht FTL — egal ob UI, CLI, Fehler, Log, man-Page
"Sieht ein Mensch den Text?" → FTL. Interner Code → kein FTL nötig
Fallback: en (immer eingebaut), weitere Sprachen als Artifacts
inotify in fs-i18n: FTL-Dateien hotplugging ohne Restart
```

### DB-Grundregeln

```
Immer über fs-db DbEngine-Trait — nie direktes SeaORM oder sqlx
Konkrete Engine kommt als Artifact (fs-db-engine-sqlite default)
```

---

## Offene Blocker

Keine offenen Blocker.

---

## Offene Architektur-Gespräche (vor Umsetzung klären)

```
[ ] G3: Bus-API Namespaces
        Welche Topics? Payloads (fs-types)? Wer darf publizieren / subscriben?

[ ] G4: CI/CD Workflow Template
        GitHub Actions für program/adapter/artifact Repos
        Wann bauen? Wie Binaries in Releases? Store-Katalog auto-update?

[ ] G5: fs-managers UI-Design
        Gemeinsames Layout via fs-render — wie sehen alle Manager gleich aus?

[ ] G6: Forks Build-Strategie
        Kanidm, Tuwunel, Stalwart, Mistral, Zentinel
        Nur kompilieren — Upstream-Sync via GitHub Actions?

[ ] G7: fs-db Design final
        DbEngine-Trait API festlegen, Migration-Strategie über alle Programme

[ ] G8: Daemon vs. Bus-Subscriber vs. Library
        Betrifft: fs-info, fs-inventory, fs-session, fs-registry
```

---

---

# Gruppe A — Neue Repos anlegen

---

## fs-bootc (program) ✅ 2026-03-30

```
Offen:
[ ] GitHub Actions: build + push zu ghcr.io/freesynergy/ bei Tag
[ ] Butane/Ignition-Config für fs-init-Integration
```

---


---

# Gruppe B — fs-libs (Monorepo, 4 Crates, kein Containerfile)

> Compile-Time Dependencies. Kein Container, kein Daemon, keine API.
> fs-libs hat 4 universelle Primitives: fs-types, fs-error, fs-crypto, fs-health

---

## fs-error (Crate in fs-libs) ✅ 2026-03-30

---

## fs-crypto (Crate in fs-libs) ✅ 2026-03-30

---

## fs-health (Crate in fs-libs) ✅ 2026-03-30

---

# Gruppe C — Infrastruktur-Libraries

---

## fs-bus (program) ✅ 2026-03-30

---

## fs-config (library, kein Container) ✅ 2026-03-30

---

## fs-db (library, kein Container) ✅ 2026-03-30

---

## fs-i18n (program) ✅ 2026-03-30

---

## fs-theme (library) ✅ 2026-03-30

---

---

# Gruppe D — GUI-Abstraktions-Layer

---

## fs-render (library, kein Container)

> GUI-Abstraktions-Traits: RenderEngine, FsWidget, FsWindow, FsTheme, FsEvent, FsView

```
OOP & Design
[x] Abstract Factory: RenderEngine erstellt Widgets/Windows
[x] FsView-Trait: view(&self) → Box<dyn FsWidget>
    WARUM in fs-render: FsWidget ist der Rückgabetyp — Trait + Rückgabetyp müssen
    im selben Crate sein. Domain-Objekte importieren fs-render NICHT direkt —
    nur ihre view.rs (Bindeglied) darf fs-render importieren.
[x] RenderEngine-Trait: create_window / run / set_context / dispatch_event / shutdown
[x] FsWidget-Trait: widget_id / is_enabled / set_enabled + render_hint / layout_hint / style_hint
[x] FsWindow-Trait: title / size / show / hide / minimize / restore / close / set_title / on_event
[x] FsEvent-Enum: Key / Mouse / Window / Custom (handle via FsWindow::on_event)
[x] AnimationSet / AnimationRegistry: load_set / activate / resolve (Registry Pattern)
[x] Immer gegen Interface — kein iced / bevy direkt in Consumer-Code

Repo
[x] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[x] Kein Containerfile (reine Library)

Code-Qualität
[x] #![deny(clippy::all, clippy::pedantic, warnings)]
[x] Keine FTL (reine Trait-Library, keine user-facing Texte)
[x] cargo clippy: 0 Fehler
[x] cargo fmt --check: sauber
[x] cargo test: 40 Tests, alle grün (alle Traits mit Stub-Impl getestet)
[x] cargo build --release: fehlerfrei

Spezifisch
[x] AppContext: Locale + Theme + FeatureFlags — durch RenderEngine::set_context durchgereicht
[x] 3D-Erweiterungs-API: Fs3dExtension + Fs3dDescriptor (optional, FsView-Basis unverändert)

Dokumentation
[x] Doku-Seite: alle Traits, FsView-Pattern (Warum hier?), AppContext, 3D-Erweiterung
[x] commit + push
```

---

## fs-gui-engine-iced (adapter) ✅ 2026-03-29

> iced-Implementierung von fs-render (libcosmic als Basis)

```
OOP & Design
[x] Adapter Pattern: IcedEngine adaptiert iced für fs-render Traits
[x] IcedEngine implementiert RenderEngine-Trait (+ set_context, run)
[x] IcedWidget implementiert FsWidget-Trait
[x] IcedWindow implementiert FsWindow-Trait
[x] IcedTheme implementiert FsTheme-Trait (iced::Theme-Wrapper)
[x] Registriert Capability "render.engine.iced" — capability.rs + CAPABILITY_ID
[x] Immer gegen Interface

Repo
[x] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[x] Containerfile

Code-Qualität
[x] #![deny(clippy::all, clippy::pedantic, warnings)]
[x] FTL-Keys für Fehlermeldungen (gui-engine-iced.ftl en + de)
[x] cargo clippy: 0 Fehler
[x] cargo fmt --check: sauber
[x] cargo test: 21 Tests grün
[x] cargo build --release: fehlerfrei

Spezifisch
[x] Elm-Pattern (MVU): mvu.rs — MvuApp<S,M,U,V> mit FsMessage-Trait
[ ] libcosmic: vollständige Integration (G2.8 — vanilla iced 0.13 als Basis)
[x] Kein iced-Code in Consumer-Crates (pub use iced als Escape-Hatch)

Dokumentation
[x] Doku-Seite: de/programme/fs-gui-engine-iced.md
[x] commit + push
```

---

## fs-gui-engine-bevy (adapter) ✅ 2026-03-30

---

## fs-web-engine (library) ✅ 2026-03-30

---

## fs-web-engine-servo (adapter) ✅ 2026-03-30

---

## fs-ui (library) — G2-Migration offen

```
Offen (G2 — nach iced-Migration):
[ ] Alle Dioxus-Komponenten auf fs-render FsView-Trait migrieren
[ ] Dioxus-Dependency vollständig entfernen
[ ] Doku-Seite: Komponenten-Übersicht, FsView-Pattern, Migration-Status
```

---

## fs-components (library) — G2-Migration offen

```
Offen (G2 — nach iced-Migration):
[ ] H5: src/nav.rs aufteilen → nav/item.rs + nav/group.rs + nav/bar.rs + nav/breadcrumb.rs
[ ] Dioxus-Dependency vollständig entfernen
[ ] Alle Komponenten auf FsView-Trait migrieren
[ ] Doku-Seite: Komponenten-Katalog, FsView-Pattern
```

---

---

# Gruppe E — Services

---

## fs-auth (program) ✅ 2026-03-30

---

## fs-registry (program) ✅ 2026-03-30

---

## fs-inventory (program) ✅ 2026-03-30

> Was ist installiert?

---

## fs-session (program) ✅ 2026-03-30

---

## fs-info (program) ✅ 2026-03-30

---

## fs-container (program) ✅ 2026-03-30

---

---

# Gruppe F — Programme

---

---

---

# Gruppe G — Apps (alle G2.9 — iced-Migration)

> Reihenfolge: klein → groß
> Jede App: UI (FsView-Trait) + CLI + API (gRPC + REST)
> Domain-Objekte importieren kein fs-render — nur view.rs als Bindeglied

---

## fs-browser (in fs-apps)

```
OOP & Design
[ ] MVC: BrowserModel / BrowserController / BrowserView
[ ] BrowserController kennt nur WebEngine-Trait + RenderEngine-Trait
[ ] NavigationHistory: Composite Pattern
[ ] BookmarkStore: Repository Pattern
[ ] Immer gegen Interface

Repo (in fs-apps/crates/fs-browser/)
[ ] CLAUDE.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Toolbar, Menü, Fehlermeldungen, Status-Texte (UI + CLI)
[ ] cargo clippy: 0 Fehler / cargo fmt / cargo test / cargo build --release

UI
[ ] iced-Migration prüfen (G2.7 abgeschlossen — vollständig?)
[ ] FsView-Trait für alle Komponenten, view.rs als Bindeglied
[ ] Servo: Feature-Flag (bis fs-web-engine-servo stabil)

CLI
[ ] fs-browser open {url} / history / bookmarks list|add|remove

API
[ ] gRPC: open / navigate / history / bookmarks
[ ] REST + OpenAPI

Dokumentation
[ ] Doku-Seite: Architektur, WebEngine-Integration, API, CLI
[ ] commit + push
```

---

## fs-theme-app (in fs-apps — heißt nur fs-theme-app weil fs-theme schon die Library ist)

```
OOP & Design
[ ] MVC + Command Pattern (Theme-Aktionen als Commands)
[ ] ThemeAppController kennt nur ThemeLoader-Trait
[ ] ThemePreview: Observer Pattern
[ ] Immer gegen Interface
[ ] FTL-Keys: alle UI + CLI Texte

UI
[ ] iced-Migration: FsView-Trait
[ ] Theme-Liste + Vorschau + Aktivieren

CLI
[ ] fs-theme list / activate {id} / preview {id}

API
[ ] gRPC: list / activate / preview
[ ] REST + OpenAPI

Dokumentation
[ ] Doku-Seite: API, CLI, ThemeLoader-Integration
[ ] commit + push
```

---

## fs-lenses (in fs-apps)

```
OOP & Design
[ ] Strategy Pattern: LensRenderer je Datentyp
[ ] LensProvider-Trait: load / render / available-lenses
[ ] LensRenderer-Trait: render(data) → FsWidget
[ ] LensRegistry: welcher Renderer für welchen Typ
[ ] Immer gegen Interface
[ ] FTL-Keys: Lens-Namen + UI-Texte

UI
[ ] iced-Migration: FsView-Trait
[ ] Lens-Auswahl + Daten-Preview

CLI / API / Doku: [ ] je fertigstellen + commit + push
```

---

## fs-ai (in fs-apps)

```
OOP & Design
[ ] Facade Pattern: AiAssistant fasst fs-llm zusammen
[ ] AiAssistant-Trait: ask / stream / context / history
[ ] ConversationStore: Repository Pattern
[ ] Kennt nur LlmAdapter-Trait — nie Mistral direkt
[ ] Immer gegen Interface
[ ] FTL-Keys: alle UI + CLI Texte

UI / CLI / API
[ ] iced-Migration: FsView-Trait (Chat-Interface + Modell-Auswahl)
[ ] CLI: fs-ai ask / stream / models
[ ] gRPC + REST + OpenAPI
[ ] DB: Konversationshistorie über fs-db DbEngine-Trait

Dokumentation: [ ] commit + push
```

---

## fs-container-app (in fs-apps)

```
OOP & Design
[ ] MVC + Command Pattern
[ ] ContainerAppController kennt nur ContainerEngine-Trait
[ ] ContainerList: Observer Pattern (Status-Updates)
[ ] Immer gegen Interface
[ ] FTL-Keys: alle UI + CLI Texte

UI / CLI / API
[ ] iced-Migration: FsView-Trait (Container-Liste + Status + Logs)
[ ] CLI: fs-container-app list / start / stop / logs
[ ] gRPC + REST + OpenAPI

Dokumentation: [ ] commit + push
```

---

## fs-tasks (in fs-apps)

```
OOP & Design
[ ] Command Pattern: jede Task-Aktion ein Kommando
[ ] TaskStore-Trait: create / update / complete / list / get
[ ] TaskTemplate-Trait: instantiate(params) → Task
[ ] DataOffer-Trait: offer / accept / list-offers
[ ] Immer gegen Interface
[ ] FTL-Keys: alle UI + CLI Texte

UI / CLI / API
[ ] iced-Migration: FsView-Trait
[ ] CLI: fs-tasks list / create / complete / delete
[ ] gRPC + REST + OpenAPI
[ ] DB: nur über fs-db DbEngine-Trait
[ ] O1: Data Offers / Accepts
[ ] O2: Task Builder UI
[ ] O3: Task-Templates aus Store

Dokumentation: [ ] commit + push
```

---

## fs-bots (program — fs-bots Repo)

```
OOP & Design
[ ] Strategy Pattern: BotAdapter je Messenger
[ ] BotAdapter-Trait: start / stop / handle-message / send
[ ] BotCommand-Trait: execute(ctx) → Result
[ ] BotRegistry: welche Bots sind aktiv
[ ] Bots als Artifacts nachladen (global oder per-Paket)
[ ] Immer gegen Interface
[ ] FTL-Keys: Bot-Antworten + Fehlermeldungen (CLI + UI)

Spezifisch
[ ] H8: bot-db/src/lib.rs aufteilen (735 Zeilen)
        → bot_db/conversation.rs + bot_db/user.rs + bot_db/state.rs + bot_db/command_log.rs
[ ] DB: nur über fs-db DbEngine-Trait

UI
[ ] iced-Migration: FsView-Trait (Bot-Übersicht + Status + Konfiguration)

CLI
[ ] fs-bots list / start / stop / status / configure

API
[ ] gRPC + REST + OpenAPI

Dokumentation
[ ] Doku-Seite: BotAdapter-Trait, BotCommand, Registry, Artifacts, API, CLI
[ ] commit + push
```

---

## fs-builder (in fs-apps)

```
OOP & Design
[ ] Pipeline Pattern: Analyse → Validierung → Build → Publish
[ ] BuildStep-Trait: execute / rollback / display_name
[ ] BuildPipeline: Chain of Responsibility
[ ] Immer gegen Interface
[ ] FTL-Keys: alle UI + CLI Texte

UI / CLI / API
[ ] iced-Migration: FsView-Trait (Build-Wizard)
[ ] CLI: fs-builder analyze / validate / build / publish
[ ] gRPC + REST + OpenAPI

Dokumentation: [ ] commit + push
```

---

## fs-store (UI-Teil in fs-apps → gehört zu fs-store Repo)

> Kein separates fs-store-app — die UI ist Teil von fs-store (Gruppe F)
> Hier nur die offenen Aufgaben die noch aus fs-apps migriert werden müssen

```
[ ] H9a: install_wizard.rs (606 Zeilen) aus fs-apps in fs-store übernehmen
         → wizard/select.rs + wizard/confirm.rs + wizard/progress.rs + wizard/done.rs
[ ] Dioxus-Code aus fs-apps/fs-store-app vollständig entfernen
[ ] commit + push
```

---

## fs-managers (program)

> Manager: Language, Theme, Icon, Cursor, ContainerApp

```
OOP & Design
[ ] Strategy Pattern: je Manager eigener Trait
[ ] LanguageManager: LanguageProvider-Trait (list / active / switch / download)
[ ] ThemeManager: kennt nur ThemeLoader-Trait
[ ] IconManager: kennt nur IconProvider-Trait
[ ] CursorManager: CursorProvider-Trait
[ ] ContainerAppManager: ContainerAppProvider-Trait
[ ] Jeder Manager: view.rs als Bindeglied zu fs-render (FsView-Trait)
[ ] Alle Manager einheitliches Aussehen via fs-render (kein Bruch)
[ ] Immer gegen Interface
[ ] FTL-Keys: alle Manager-Texte (UI + CLI + API)

Spezifisch — große Dateien aufteilen
[ ] H9b: language_panel.rs (1060 Zeilen)
         → language/list.rs + language/download.rs + language/active.rs + language/preview.rs
[ ] H9c: cursor_panel.rs (808 Zeilen)
         → cursor/list.rs + cursor/preview.rs + cursor/active.rs
[ ] H9d: manager_view.rs (792 Zeilen)
         → views/language_view.rs + views/theme_view.rs + views/icon_view.rs + …

Bekannter Bug
[ ] fs-manager-language: alte gix-API (pre gix 0.65)
        prepare_push + SignatureRef auf neue gix-API migrieren

UI
[ ] iced-Migration: alle Manager-Views als FsView-Trait
[ ] G5: gemeinsames Layout-Muster (nach Architektur-Gespräch)

CLI
[ ] fs-managers language list|set|download
[ ] fs-managers theme list|set
[ ] fs-managers icons list|set
[ ] fs-managers cursor list|set

API
[ ] gRPC je Manager + gemeinsamer REST-Endpoint
[ ] OpenAPI: auto-generiert

Dokumentation
[ ] Doku-Seite: Manager-Übersicht, je Manager: Trait + API + CLI, FsView-Pattern
[ ] commit + push
```

---

---

# Gruppe H — Store/ Katalog

---

## Store/ (Metadaten-Katalog — kein Container, kein Rust-Code) ✅ 2026-03-30

> Nur TOML-Dateien: package.toml je Paket, bundle.toml je Bundle

```
Offen:
[ ] README.md: Katalog-Struktur erklären
[ ] Alle bestehenden Pakete auf einheitliches Format migrieren (low priority)
```

---

---

# Gruppe I — Dokumentation

---

## fs-documentation ✅ 2026-03-30

---

---

# Langfristig

---

## Search (Phase M)

```
M1. [ ] Search-View (Suchfeld, gruppierte Ergebnisse, Preview)
M2. [ ] Service-Suche (lokal, fs-registry)
M3. [ ] Host-Suche (Bus-aggregiert)
M4. [ ] Föderale Suche
```

## Federation (Phase P)

```
P2. [ ] Federation-Grundstruktur
P3. [ ] Rechte-Kaskade + Audit-Log
P4. [ ] Föderaler Bus
```

## Mail (Phase R)

```
R1. [ ] Stalwart als App-Paket (Store-Eintrag, Binary aus Fork)
R2. [ ] IAM-Integration (Kanidm via OIDC/LDAP)
R3. [ ] Adapter (smtp, imap → fs-registry)
R4. [ ] Domain-Konfiguration pro Node
```

## Kontakte & Kalender (Phase S)

```
S1. [ ] Rustical evaluieren (CalDAV + CardDAV)
S2. [ ] Kontakte-Backend (vCard 4.0)
S3. [ ] Kalender-Backend (iCal/CalDAV)
S4. [ ] IAM-Integration
S5. [ ] App-Pakete: contacts-server, calendar-server
```

## Infrastruktur (Phase T)

```
T1. [ ] Vaultwarden
T2. [ ] Ntfy / UnifiedPush
T3. [ ] Element Call + coturn
T4. [ ] WireGuard (nach Federation)
T5. [ ] Hickory DNS (nach Federation)
```

## Polish (Phase Q)

```
Q1. [ ] Action Registry + konfigurierbare Shortcuts
Q2. [ ] Auto-generierte Shortcut-Referenz
Q3. [ ] Menü: jeder Punkt ruft echte Aktion auf
Q4. [ ] Profil: IAM + editierbar + Account-Linking
Q5. [ ] Notification Bell
Q6. [ ] Context-Menüs
Q7. [ ] Animationen konfigurierbar (AnimationSet aus Store)
Q8. [ ] Alle Stubs / toten Code entfernen
```

## matrix-sdk State Store

```
N0. [ ] PostgreSQL State Store (Blocker: fs-db-engine-postgres fertig)
        Bug: matrix feature → recursion overflow in rustc ≥1.94 (upstream)
```

---

## Reihenfolge

```
1.  Blocker klären (D24, H2)
2.  Architektur-Gespräche (G3, G7, G8)
3.  Neue Repos anlegen (fs-bootc)
4.  Store/ Katalog: package.toml + bundle.toml Format + Bundles anlegen
5.  fs-db: DbEngine-Trait final
6.  fs-i18n: inotify + OCI Artifacts + Global vs. Per-Paket
7.  fs-bus: gRPC-API + Namespaces (G3)
8.  Services: fs-auth → fs-registry → fs-inventory → fs-session → fs-info
9.  GUI: fs-render Doku → fs-gui-engine-iced → fs-gui-engine-bevy
10. G2.9: Apps iced-Migration (browser → theme-app → lenses → ai →
          container-app → tasks → bots → builder → managers)
11. Große Dateien aufteilen (H-Tasks)
12. fs-desktop: Wayland-Compositor (langfristig)
13. Architektur-Gespräche (G4, G5, G6)
14. Langfristig: M, P, R, S, T, Q
```
