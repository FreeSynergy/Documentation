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
Immer über fs-db Repository<T> + Filter<T> — kein SQL-String im Consumer-Code
Konkrete Engine kommt als Artifact (fs-db-engine-sqlite default)
DbEngine-Trait + direktes SeaORM/sqlx nur in Adapter-Repos (fs-db-engine-*)
```

---

## Offene Blocker

Keine offenen Blocker.

---

## Offene Architektur-Gespräche (vor Umsetzung klären)

Keine offenen Architektur-Gespräche.

> G3–G8 geklärt 2026-03-31 — dokumentiert in:
> - G3: konzepte/bus-api-namespaces.md (Event Topics + Namespaces)
> - G4: technik/build-workflow.md (Plattformen + CI reusable workflows)
> - G5: konzepte/manager.md (ManagerLayout-Trait, kein Master-Manager)
> - G6: technik/forks-build-strategie.md (OCI-only, Upstream-Sync)
> - G7: technik/fs-db.md (Repository<T> + Filter<T>, kein Raw SQL)
> - G8: konzepte/event-driven-services.md (Observer via Bus, Startup-Reihenfolge)

---

---

# Gruppe A — Neue Repos anlegen

---

## fs-bootc (program) ✅ 2026-03-31

```
GitHub Actions build.yml: baut fs-server + fs-workstation OCI bei push/tag, pushes zu ghcr.io/freesynergy/.
Butane/Ignition: ignition/fs-server.bu + ignition/fs-workstation.bu (fsadmin, node.toml, init.toml, systemd).
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

## fs-bus (program) ✅ 2026-03-31

```
G3 erledigt: topics.rs (20 Konstanten, ::‑Separator), topic_matches auf :: umgestellt,
alle Tests aktualisiert, bus_wiring.rs + e2e_install.rs verwenden Konstanten.
```

---

## fs-config (library, kein Container) ✅ 2026-03-30

---

## fs-db (library, kein Container) ✅ 2026-03-31

```
Offen:
[ ] CrudRepo auf neues Repository<T> migrieren oder ablösen (low priority)
[ ] SqliteEngine + PostgresEngine: Filter<T> → SQL übersetzen (im jeweiligen Adapter-Repo, nach G8)
```

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
```

---

## fs-components (library) — G2-Migration offen

```
Offen (G2 — nach iced-Migration):
[ ] H5: src/nav.rs aufteilen → nav/item.rs + nav/group.rs + nav/bar.rs + nav/breadcrumb.rs
[ ] Dioxus-Dependency vollständig entfernen
[ ] Alle Komponenten auf FsView-Trait migrieren
```

---

---

# Gruppe E — Services

---

## fs-auth (program) ✅ 2026-03-30

---

## fs-registry (program) ✅ 2026-03-31

```
G8 erledigt: RegistryBusHandler in MessageBus eingehängt, startup::registered + shutdown::stopped
publishen, 2 neue Bus-End-to-End-Tests, gRPC schon vollständig (Register/Deregister/List/Lookup/EndpointFor/SetStatus/Health).
```

---

## fs-inventory (program) ✅ 2026-03-31

```
G8 erledigt: InventoryBusHandler auf inventory::#, upsert_resource aktualisiert Version+Paths,
4 neue Bus-Tests (install/update/remove via event, topic_pattern).
gRPC: list_installed/get_package/is_installed bereits vollständig.
```

---

## fs-session (program) ✅ 2026-03-31

```
G8 erledigt: SessionBusHandler implementiert (login/logout/app::opened/app::closed),
Daemon mit gRPC (CurrentUser/OpenApps/SessionInfo/Health), SessionStore-Trait + SQLite-Impl,
4 Bus-Tests + 9 Store-Tests, alle grün.
```

---

## fs-info (program) ✅ 2026-03-31

```
G8 erledigt: AlertPublisher (degraded/restored via Bus), GrpcInfo (SystemInfo/CpuUsage/MemoryInfo/DiskInfo/Health),
CLI (Daemon/System/Cpu/Memory/Disk/Alerts --monitor), proto/info.proto, build.rs, [[bin]] fs-info.
```

---

## fs-container (program) ✅ 2026-03-30

---

---

# Gruppe F — Programme

---

---

---

# Gruppe G — Apps (eigene Repos — G2.9 iced-Migration ausstehend)

> Jede App hat ein eigenes Repo. fs-apps archiviert (2026-04-01).
> Jede App: UI (FsView-Trait) + CLI + API (gRPC + REST)
> Domain-Objekte importieren kein fs-render — nur view.rs als Bindeglied

---

## fs-browser ✅ 2026-04-01

```
MVC (BrowserModel/Controller/View), FsView-Trait in view.rs, BookmarkStore-Trait,
NavigationHistory, keys.rs FTL-Konstanten, CLI (open/history/bookmarks), gRPC, REST+OpenAPI.
Dioxus vollständig entfernt. Servo: Feature-Flag vorhanden (wartet auf fs-web-engine-servo).
Offen: Doku-Seite (niedrige Prio)
```

---

## fs-theme-app ✅ 2026-04-01

```
Erledigt: ThemeController (MVC), gRPC (list/activate/preview/health), REST+OpenAPI,
CLI (list/active/activate/preview/daemon), FsView-Trait (view.rs), build.rs, proto/theme_app.proto.

Offen (G2 — iced-Migration):
[ ] Dioxus komplett entfernen (app.rs ersetzen durch iced-Engine)
[ ] FTL-Keys: alle UI + CLI Texte in .ftl migrieren (derzeit .toml-Snippets)
[ ] Doku-Seite + commit + push
```

---

## fs-lenses ✅ 2026-04-01

```
Erledigt: LensController (Strategy Pattern), gRPC, REST+OpenAPI, CLI, FsView-Trait, build.rs.

Offen (G2 — iced-Migration):
[ ] Dioxus komplett entfernen
[ ] FTL-Keys migrieren
[ ] LensRegistry: welcher Renderer für welchen Typ (Strategy Pattern vollständig)
[ ] Doku-Seite + commit + push
```

---

## fs-ai ✅ 2026-04-01

```
Erledigt: AiController (Facade über fs-manager-ai), gRPC (list/status/start/stop/health),
REST+OpenAPI, CLI (models/status/start/stop/daemon), FsView-Trait (view.rs), build.rs.

Offen (G2 — iced-Migration):
[ ] Dioxus komplett entfernen
[ ] FTL-Keys migrieren
[ ] DB: Konversationshistorie über fs-db DbEngine-Trait
[ ] Doku-Seite + commit + push
```

---

## fs-container-app ✅ 2026-04-01

```
Erledigt: ContainerAppController<E: ContainerEngine> (MVC, kennt nur Trait), gRPC (list/start/stop/health),
REST+OpenAPI, CLI (list/start/stop/daemon), FsView-Trait (view.rs), build.rs, proto/container_app.proto.

Offen (G2 — iced-Migration):
[ ] Dioxus komplett entfernen
[ ] FTL-Keys migrieren
[ ] Doku-Seite + commit + push
```

---

## fs-tasks ✅ 2026-04-01

```
Erledigt: TaskController (Command Pattern), gRPC (list/create/delete/toggle/health),
REST+OpenAPI, CLI (list/create/delete/toggle/daemon), FsView-Trait (TasksView/TaskDetailView/CreateTaskView),
build.rs, proto/tasks.proto. 9 Tests grün.

Offen (G2 — iced-Migration):
[ ] Dioxus komplett entfernen
[ ] FTL-Keys migrieren
[ ] DB: TaskStore über fs-db DbEngine-Trait
[ ] O1: Data Offers / Accepts
[ ] O2: Task Builder UI
[ ] O3: Task-Templates aus Store
[ ] Doku-Seite + commit + push
```

---

## fs-bots ✅ 2026-04-01

```
Erledigt: BotController (Strategy Pattern via bot_strategy), gRPC (list/get/enable/disable/health),
REST+OpenAPI, CLI (list/enable/disable/daemon), FsView-Trait (view.rs), build.rs, proto/bots.proto.
13 Tests grün (inkl. BotStrategy + Platform).

Offen (G2 — iced-Migration):
[ ] Dioxus komplett entfernen
[ ] FTL-Keys migrieren
[ ] H8: bot-db/src/lib.rs aufteilen (735 Zeilen)
        → bot_db/conversation.rs + bot_db/user.rs + bot_db/state.rs + bot_db/command_log.rs
[ ] DB: nur über fs-db DbEngine-Trait
[ ] Doku-Seite + commit + push
```

---

## fs-builder ✅ 2026-04-01

```
Erledigt: BuilderController (Pipeline Pattern: Analyse→Validate→Build→Publish), BuildStep-Trait,
BuildPipeline (Chain of Responsibility), gRPC (status/health), REST+OpenAPI, CLI (status/daemon),
FsView-Trait (view.rs), build.rs, proto/builder_app.proto. 9 Tests grün.

Offen (G2 — iced-Migration):
[ ] Dioxus komplett entfernen
[ ] FTL-Keys migrieren
[ ] CLI: fs-builder analyze / validate / build / publish (vollständige Pipeline-Steuerung)
[ ] Doku-Seite + commit + push
```

---

## fs-store (UI-Teil in fs-store Repo) ✅ H9a 2026-04-01

> Kein separates fs-store-app — die UI ist Teil von fs-store (Gruppe F)

```
Erledigt H9a: installer/ Modul in fs-store (InstallRequest, InstallKind,
download-Funktionen für Language/Theme/Container/App-Binary, env-var-Extraktion).
wizard/done.rs in fs-store-app (InstallResult + FsWidget). 10 Tests grün.

Offen (H9b–H9d):
[ ] wizard/select.rs — Paketauswahl UI
[ ] wizard/confirm.rs — Bestätigungsschritt
[ ] wizard/progress.rs — Fortschrittsanzeige
```

---

## fs-managers ✅ 2026-04-01

```
Erledigt: ManagersController (MVC, Arc-backed), ManagerKind-Enum (Language/Icons/Cursor/Theme/Container),
gRPC (ListPanels/GetStatus/Health), REST+OpenAPI, CLI (list/status/daemon), FsView-Trait (view.rs),
build.rs, proto/managers.proto. 7+12 Tests grün. Dioxus vollständig entfernt.
H9b/H9c: language + cursor in Sections aufgeteilt (list/active/preview/download).
FsView + ManagerLayout auf allen 5 Managern implementiert.

Offen (Langfristig):
[ ] Bekannter Bug: fs-manager-language: gix-API (pre-0.65) in git.rs migrieren
[ ] FTL-Keys migrieren
[ ] Doku-Seite + commit + push
```

---

---

# Gruppe H — Store/ Katalog

---

## Store/ (Metadaten-Katalog — kein Container, kein Rust-Code) ✅ 2026-03-31

> Nur TOML-Dateien: package.toml je Paket, bundle.toml je Bundle

```
Offen:
[ ] Alle bestehenden Pakete auf einheitliches Format migrieren (low priority)
```

---

---

# Gruppe I — Dokumentation

---

## fs-documentation ✅ 2026-03-30

---

---

# Gruppe J — CI/CD (G4)

---

## fs-ci (neues Repo — reusable workflows) ✅ 2026-03-31

```
ci-check.yml (Clippy+fmt+test+deny+audit) + release-desktop.yml (Linux x2 + Win x2 + macOS + OCI).
Offen: release-mobile.yml (Android + iOS — separate Phase)
```

---

## GitHub Actions — alle bestehenden Repos nachrüsten ✅ 2026-03-31

```
19 Repos mit .github/workflows/ci.yml + release.yml (außer fs-render + fs-libs: nur ci.yml).
Alle committed + gepusht.
```

---

---

# Gruppe K — Forks (G6)

---

## Fork-Repos: Containerfiles + CI ✅ 2026-03-31

```
Alle 6 Forks: Containerfile + sync-upstream.yml + fsn-build.yml + release.yml (OCI via fs-ci/release-oci.yml).
Store-Katalog: type="fork" + upstream-Link in allen 6 catalog.toml.
fs-ci: release-oci.yml reusable workflow hinzugefügt.
```

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
N0. [ ] PostgreSQL State Store (Blocker aufgelöst: fs-db-engine-postgres ✅ 2026-03-31)
        Bug: matrix feature → recursion overflow in rustc ≥1.94 (upstream, noch offen)
```

---

## Reihenfolge

```
1.  fs-ci Repo anlegen (Gruppe J) — Basis für alle weiteren CI/CD-Tasks
2.  fs-bus: G3 — Event-Topic-Konstanten + Payload-Structs in fs-types
3.  fs-db: G7 — Repository<T> + Filter<T> implementieren
4.  Services G8: fs-registry → fs-inventory → fs-session → fs-info
    (Event-Driven Integration, Bus-Subscribe + Publish)
5.  GitHub Actions für alle bestehenden Repos nachrüsten (Gruppe J)
6.  Fork-Repos: Containerfiles + CI (Gruppe K, Reihenfolge: kanidm → stalwart → tuwunel → mistral → zentinel)
7.  fs-bootc: GitHub Actions + Butane/Ignition-Config
8.  G2.9: Apps iced-Migration (browser → theme-app → lenses → ai →
          container-app → tasks → bots → builder → managers)
9.  fs-managers G5: ManagerLayout-Trait + alle Manager-Views
10. Große Dateien aufteilen (H-Tasks)
11. Langfristig: M, P, R, S, T, Q
```
