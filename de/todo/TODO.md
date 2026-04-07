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
Claude verwendet dieselben Flags wie der Hook — nie abweichende Konfigurationen.

---

## Architektur-Überblick (Stand 2026-04-05)

### Grundprinzipien

```
Standalone-First:
  Jedes Repo / jeder Container funktioniert unabhängig vom Rest.
  Integration über gRPC / REST / Bus — nie harte Runtime-Dependencies.

Everything through the Store:
  Was nicht im Store ist, existiert für das System nicht.
  Pakete, Engines, Themes, Sprachen, Adapter — alles ist ein Store-Artifact.

Engine-Auswahl zur Laufzeit:
  fs-info ermittelt Capabilities (display_server, terminal, headless).
  Render-Engine wird als Artifact aus dem Store geladen.
  display_server → GUI (iced oder bevy — User wählt)
  nur terminal   → TUI
  headless / SSH → API + CLI only

Trait-First:
  Traits definieren — alle Engines (iced, TUI, bevy) implementieren sie.
  Engine-Wechsel = nur neue Impl, kein App-Code-Wechsel.
  Domain-Objekte importieren KEIN fs-render.
  view.rs ist das einzige Bindeglied zwischen Domain + Rendering.

Activity-First (NEU):
  Der Mensch denkt in Absichten, nicht in Programmen.
  Activity Hub zeigt Kategorien — "was will ich tun?" statt "welches Programm?"
  Jede Activity kann von einem beliebigen Programm angeboten werden
  (via ActivityEngine-Trait + Adapter).
```

### API-Standard

```
gRPC (tonic):     primär — intern + extern, type-safe
REST (axum):      zusätzlich — universell, Browser-kompatibel
OpenAPI (utoipa): auto-generiert
NIE direkte Calls über Repo-Grenzen — immer über gRPC/REST
serde/JSON nur bei externen APIs (3rd-party Forks: Kanidm, Forgejo, ...)
```

### Package-Typen im Store

```
bundle   — Gruppe von Paketen (z.B. Workstation, Server, Minimal)
program  — Laufender Dienst (Container bevorzugt)
adapter  — Implementiert Trait, registriert Capability (kommt mit program)
artifact — Reine Daten (Sprachen, Themes, DB-Engines, Render-Engines, WASM-Komponenten)
fork     — 3rd-party Fork (Kanidm, Tuwunel, Stalwart, Zentinel, ...)
```

### Bundle-Definitionen

```
Minimal:     fs-init + fs-store + fs-registry + fs-inventory
Server:      Minimal + fs-auth(Kanidm) + fs-node + Zentinel + Stalwart
Workstation: Server + Desktop + render-engine(User-Wahl) + fs-managers + Apps
Developer:   Workstation + Forgejo + erweiterte Manager-Tools
```

### OOP-Grundregeln

```
Design Pattern ZUERST — vor dem ersten Code
Traits + Structs zuerst, dann Impl
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

## Abgeschlossene Phasen (kompakt)

```
Phase 1 ✅  Bootstrap: fs-init (Capability-Detection, Install-Wizard, Standalone-Test)
Phase 2 ✅  Store: Katalog, Install-Pipeline (7 Steps), CLI install/remove/update, Bundles
Phase 3 ✅  fs-render: LayoutDescriptor, WASM-Sandbox, iced+TUI Engines, 5 Standard-Komponenten
Phase 4 ✅  Desktop: Shell-Layout, Komponenten-Registry, App-Management, Settings Layout-Seite, i18n
Phase 4B ✅ Desktop Visual/UX: Sidebar State Machine, SVG-Icons, Help+AI-Sidebar, HelpSource-Trait
Phase 5.1 ✅ Kanidm: 4 Protokoll-Traits, KanidmSetupWizard, Desktop-Login
Phase 5.2 ✅ Zentinel: ZentinelManager (Facade), RouteConfig, Auto-Routing via Bus
Phase 5.3 ✅ Stalwart: SmtpProvider+ImapProvider+JmapProvider, StalwartSetupWizard
Phase 5.4 ✅ Tuwunel: MatrixBotAdapter, TuwunelSetupWizard
Phase 5.5 ✅ Telegram: TelegramAdapter, BotChannel-Trait, fs-manager-telegram
Phase 5.X ✅ fs-pod-forge: PodConfigurator-Trait, ManifestDiff, Systemd-Units
Phase 5.X+1 ✅ fs-app-forge: AppConfigurator-Trait, Schema-driven UI, Atomic Write
Phase 5.X+2 ✅ Manager-Upgrade: ServiceController + CategoryManager Traits
Phase 5.6 ✅ Forgejo: GitProvider-Trait, ForgejoSetupWizard, ForgejoCategoryController
Phase 5.7 ✅ Outline + Wiki.js: WikiProvider-Trait (Strategy), beide Adapters
Phase 5B ✅ Store: PackageCatalogValidator, Tag-Filter, Updates-Tab
Phase 6.1 ✅ Apps: LensRegistry, TaskStore, ConversationStore (alle Strategy Pattern)
Phase 6.2 ✅ Search: SearchStrategy (Demo + Bus), SearchView, LensQueryEngine
Phase 7 ✅  Federation: Rechte-Kaskade, AuditLog, FederationEvent Bus (7 Topics)
           Infrastruktur: Vaultwarden, Ntfy, Element Call im Store
G1.1  ✅  Navigations-Traits (fs-render): Corner/Side/HoverMag/ProgramView (2026-04-05)
G1.4  ✅  Program-Modell: caption, ProgramGroup, CompositeIcon-Felder, ProgramViewProvider (2026-04-06)
G1.7  ✅  Content-Komponenten: ExpandableGroup, HelpSource, FocusObserver, GeneralHelp, FocusHelp, InventoryList (2026-04-06)
           SettingsConfigComponent (fs-settings), SettingsContainerComponent (fs-container-app)
           AiComponent (fs-ai), SearchComponent (fs-gui-workspace)
G1.9  ✅  UX-Extras (2026-04-06): BadgedIcon, WindowLayoutMode (Normal/Tiling/Focus),
           QuickSwitchCommand, NotificationCenter, WorkspaceProfileManager, TouchHandler,
           PinboardStore, AutoThemeSchedule, BreadcrumbProvider, ClipboardHistory,
           WindowSnapManager — fs-render + fs-components + fs-gui-workspace + i18n (40 Keys)
```

---

---

# G1 — Desktop-Neustrukturierung

> Das Desktop-Paradigma wechselt grundlegend:
> Weg von Programm-zentrischen Sidebars, hin zu Absichts-zentrischen Navigationsmenüs.
> Activity Hub ersetzt den "Category Starter" — der Mensch denkt in Absichten, nicht in Programmen.
> Alle Navigations-Komponenten werden als Traits definiert — alle Engines (iced, TUI, bevy)
> implementieren sie. Engine-Wechsel = keine App-Code-Änderung.

---

## G1.1 — Navigations-Traits (fs-render) ✅ 2026-04-05

Implementiert in `fs-render/src/navigation.rs` (90 Tests grün, clippy + fmt sauber).
- `Corner`, `Side`, `IndicatorStyle`, `Distribution`, `IconRef`, `CompositeIcon`
- `MenuItemDescriptor` (beliebig tief schachtelbar)
- `CornerMenuDescriptor` trait, `SideMenuDescriptor` trait
- `HoverMagnification` trait (`size_at_distance()` mit exponentialem Falloff)
- `ProgramView` enum, `ProgramViewProvider` trait
- i18n: `navigation.ftl` (en + de)
- Doku: `konzepte/navigation-menus.md` + `konzepte/program-views.md` aktualisiert

## G1.2 — Navigation: iced-Implementierung (fs-gui-engine-iced) ✅ 2026-04-05

Implementiert in `fs-gui-engine-iced/src/navigation.rs` (18 Tests grün, clippy + fmt sauber).
- `MenuConfig` — icon_size, max_icon_size, spread, indicator_radius, accent
- `CornerMenuState` / `SideMenuState` — MVU-kompatibel (open, hovered_idx)
- `NavMessage` enum — Toggle, ItemEntered, ItemLeft, Action für Corner + Side
- `update_corner_menu()` / `update_side_menu()` — State-Updater (Dispatcher-freundlich)
- Indikatoren: asymmetrische border-radius → Viertelkreis (Corner) / Halbkreis (Side)
- Items: Column von transparenten Buttons; Höhe skaliert per Hover-Magnification
- Sub-Items: ▶-Suffix auf label_key (Basis für Sub-Ebenen in G1.5)
- Scroll-Fallback: `scrollable` ab SCROLL_THRESHOLD = 8 Items
- Icon-Hintergründe: `background: None` → immer transparent
- keys.rs: NAV_CORNER_INDICATOR, NAV_SIDE_INDICATOR, NAV_ITEM_HAS_SUB
- Re-exports in lib.rs

## G1.3 — Navigation: Komponenten (fs-components) ✅ 2026-04-05

Implementiert in `fs-components/src/navigation_menu.rs` (20 Tests grün, clippy + fmt sauber).
- `CornerMenuConfig` + `CornerMenu` + `CornerMenuWidget` (FsView + CornerMenuDescriptor + HoverMagnification)
- `SideMenuConfig` + `SideMenu` + `SideMenuWidget` (FsView + SideMenuDescriptor + HoverMagnification)
- `SidebarPanelConfig` + `SidebarPanel` + `SidebarPanelWidget` (FsView, Overlay-Sidebar, kein Content-Shift)
- Alle drei mit `scroll_fallback` für kleine Screens / Mobile
- i18n: `navigation.ftl` (en + de) um `nav-menu-style-*`, `nav-sidebar-panel-*`, `nav-scroll-more-items` ergänzt

## G1.4 — Program-Modell-Erweiterungen (fs-inventory + fs-store) ✅ 2026-04-06

```
caption in InstalledResource + PackageData, ProgramGroup struct,
CompositeIcon-Felder (secondary_icon_path, overlap_factor),
ProgramViewProvider impl für Kanidm/Stalwart/Tuwunel/Zentinel/Forgejo/Wiki/Desktop,
InfoViewScope (own-machine vs remote-server), i18n inventory.ftl (en+de),
Doku: konzepte/program-views.md
```

## G1.5 — Desktop-Shell-Refactor (fs-gui-workspace) ✅ 2026-04-06

```
Umgesetzt in fs-desktop/crates/fs-gui-workspace/src/:
- capability_observer.rs (neu): CapabilityObserver — env-var-Heuristik, G1.7: gRPC
- corner_menus.rs (neu): TasksMenu (TL), SettingsMenu (BL), HelpMenu (TR), AiMenu (BR)
- shell.rs: Sidebars entfernt → 4 Corner-Menu-Overlays als stack-Layer
- shell.rs: Wallpaper (WallpaperSource::Color → iced bg-Color)
- shell.rs: Titlebar + View-Buttons (Start/Info/Manual/Settings) + Tiling-Toggle
- Neben-Fix fs-render: SlotKind::Sidebar Variante
- Neben-Fix fs-gui-engine-iced: render_element() für ExpandableGroup/TextInput/SearchResult
- fs-i18n desktop.ftl (en+de): Corner-Menü-, Titlebar-, Appearance-Keys
- 48 Tests grün, clippy + fmt sauber, 4 Repos committed + gepusht
```

## G1.6 — Activity Hub (fs-components + eigene Repos) ✅ 2026-04-06

Implementiert in `fs-render/src/activity.rs` + `fs-components/src/activity_hub.rs`
(105+85 Tests grün, clippy + fmt sauber).
- `ActivityAction` (New/Open/Recent/Browse/Custom) + `label_key()`
- `ActivityCategory` (6 Kategorien + Custom) + `label_key()`
- `ActivityEngine` trait: `activity_id`, `activity_name_key`, `supported_actions`,
  `default_view`, `category`, `can_be_default`
- `Activity` struct: id, icon, name_key, category, `engine: Option<Arc<dyn ActivityEngine>>`,
  `sub_activities: Vec<Activity>`; Konstruktoren `new()` + `container()`
- `ActivityHub` Registry: `register`, `all`, `by_category`, `search`, `to_menu_items`;
  `view_mode: ActivityHubView` (CornerMenuEntry | DesktopWidget | Both)
- Widget-Typen: `ActivityWidget` (CSS `fs-activity`) + `ActivityHubWidget` (CSS `fs-activity-hub`)
- i18n: `activity-hub.ftl` (en + de) in fs-i18n
- Doku: `konzepte/activity-hub.md` vollständig
- Offen (nach G1.4/G1.5): Activity-Views UI, fs-inventory gRPC Filter, Adapter-Impls

## G1.7 — Content-Komponenten ✅ 2026-04-06

```
Design Pattern: Strategy (HelpSource) + Observer (FocusObserver) + Facade (SettingsHub)

Implementiert:
- InventoryListComponent (fs-render): ProgramGroup + ExpandableGroup + CompositeIcon
- GeneralHelpComponent (fs-render): context_key + action_keys via ComponentCtx.config
- FocusHelpComponent (fs-render): focus_element_id + focus_help_key via ComponentCtx.config
- HelpSource trait (fs-render): context_key() + action_keys() — Screen deklariert Kontext
- FocusObserver trait (fs-render): on_focus() + focused_element()
- ExpandableGroup / TextInput / SearchResult Varianten in LayoutElement
- SettingsConfigComponent (fs-settings): 4 Sektionen + Custom per program_id
- SettingsContainerComponent (fs-container-app): Pod-YAML + Berechtigungscheck + Restart-Badge
- AiComponent (fs-ai): Chat mit Capability-Guard (ai_chat_available)
- SearchComponent (fs-gui-workspace): Input + Filter (Strategy) + Ergebnisse nach Quelle
- i18n: render.ftl (en+de) + ai.ftl (en+de) erweitert
- Alle Repos committed + gepusht
```

## G1.8 — Offene Items (konsolidiert aus G0)

```
Settings-Seiten (fs-settings):  ← Appearance + Language + ServiceRoles + Packages ✅ 2026-04-06
[ ] Accounts-Settings: IAM via Kanidm (OIDC-Login-Flow)
[ ] Shortcuts-Settings: Tastaturkürzel-Editor

Desktop-Shell (fs-gui-workspace):  ← HotReload + IcedLayoutInterpreter ✅ 2026-04-06
    AiHelpSource: REST → POST /ai/chat ✅ 2026-04-06

Store (fs-store + Store/):  ← Screenshots + BrowseMode + Install-Button ✅ 2026-04-06
    catalog.toml: zentinel, telegram, desktop, init, store, auth, managers, Render-Engines, DB-Engines ✅ 2026-04-07
    Doku: technik/store-catalog-spec.md ✅ 2026-04-07

Datenbank (fs-db):  ← CrudRepo entfernt + DbRecord für 9 Entities ✅ 2026-04-06
    Filter<T> → find_filtered() in allen 9 Repos via SeaORM raw SQL ✅ 2026-04-07

Bots (fs-bots):  ← bot-db auf DbEngine-Trait migriert ✅ 2026-04-07

Search (fs-lenses + fs-search):  ← BusSearchStrategy (wire+collect via mpsc+timeout) ✅ 2026-04-07
[ ] Service-Suche: lokal via fs-registry
[ ] Host-Suche: Bus-aggregiert via BusSearchStrategy
[ ] Föderale Suche
[ ] i18n: ALLE Such-Texte in FTL

Manager-Integration:
[ ] UpdatePodConfig: ManagerAction::UpdatePodConfig → PodConfigurator::apply()
    → podman play kube --replace <generated.yml>
[ ] EditConfig: ManagerAction::EditConfig → AppConfigurator::apply()
[ ] Role-Switching: fs-registry Capability-Eintrag umschreiben wenn set_active() aufgerufen
[ ] Update-Check: update_available() mit echtem Store-Lookup implementieren

Forgejo (fs-managers):
[ ] Store-Eintrag: forgejo catalog.toml + pod.yml
[ ] S3: Repository-Storage via opendal
[ ] ForgejoAdapter → fs-registry wiring (Service Role: git)

Standalone-Tests (Container laufen muss):
[ ] Kanidm-Integration-Test (env vars setzen)
[ ] Zentinel-Integration-Test (env vars setzen)
[ ] Stalwart-Integration-Test (env vars setzen)
[ ] Tuwunel-Integration-Test (env vars setzen)
[ ] Telegram-Test (echtes Bot-Token)
[ ] gRPC subscribe Telegram: Streaming-Impl (aktuell REST long-poll)

Technisches Debt (wartet auf Upstream):
[ ] matrix-sdk live Feature: rustc ≥1.94 Fix (matrix-sdk 0.16)
[ ] matrix-sdk PostgreSQL State Store (recursion-overflow Fix upstream)
[ ] Zentinel S3-Integration: opendal für Konfigurations-Storage
[ ] JMAP-Login-Flow in fs-settings konfigurierbar
```

## G1.9 — UX-Extras ✅ 2026-04-06

Vollständig implementiert — siehe Abgeschlossene Phasen oben.
```

---

---

# Reihenfolge

```
--- Aktuell (G1) ---

G1.1  Navigations-Traits (fs-render)       ← Fundament für alle Navigation
G1.2  iced-Implementierung                 ← erste sichtbare Engine
G1.3  Navigations-Komponenten (fs-components)
G1.4  Program-Modell (fs-inventory + fs-store)
G1.5  Desktop-Shell-Refactor              ← nutzt G1.1–G1.3
G1.6  Activity Hub                        ← nutzt G1.1–G1.3 + G1.4
G1.7  Content-Komponenten                 ← parallel zu G1.5
G1.8  Offene Items konsolidiert           ← parallel wenn möglich
G1.9  UX-Extras ✅                        ← abgeschlossen 2026-04-06

--- Langfristig (G1+) ---

G1+   ActivityPub Federation (7.1 Grundstruktur → echte AP-Impl)
G1+   WireGuard (nach AP-Grundstruktur)
G1+   Hickory DNS (nach AP-Grundstruktur)

--- G2 ---

G2    libcosmic: vollständige Integration in fs-gui-engine-iced
G2    fs-gui-engine-tui: Navigation-Traits implementieren (Corner/Side Menu in TUI)
G2    fs-gui-engine-bevy: Navigation-Traits implementieren (3D-fähig)
G2    fs-render + iced/bevy: Dioxus komplett raus (G2 plan)
G2    ProgramView::Binding — Workflow-Editor (Programme verknüpfen)
```

---

## Archiviert (Referenz)

```
fs-apps:    archiviert 2026-04-01 (alle Apps in eigene Repos migriert)
fs-builder: archiviert 2026-04-02 (Funktionalität in fs-managers; Pipeline-Pattern erhalten)
CHANGELOG.md: nicht mehr gepflegt (seit 2026-03)
```
