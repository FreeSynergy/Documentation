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

## G1.2 — Navigation: iced-Implementierung (fs-gui-engine-iced)

```
Design Pattern: Interpreter (LayoutDescriptor → iced Element)
Primär: iced (G1). TUI + bevy folgen automatisch (gleiche Traits, separater Durchgang).

[ ] CornerMenu rendering:
      Viertelkreis-Indikator (immer sichtbar — zeigt wo das Menü ist)
      Radiales Aufklappen: Items erscheinen auf Kreisbogen
      Standardgröße: konfigurierbar via DesktopConfig (Settings > Ansicht)

[ ] SideMenu rendering:
      Halbkreis-Indikator (immer sichtbar — zeigt wo das Menü ist)
      Accordion-Animation: Items klappen von der Mitte nach oben+unten auf
      Gleichmäßige Verteilung wenn wenige Items (muss Rand nicht berühren)

[ ] HoverMagnification:
      SVG-Icons skalieren verlustfrei → HoverMagnification kein Problem
      Nachbar-Icons vergrößern sich proportional

[ ] Sub-Menüs: Item-Klick öffnet neue Ebene (weitere Kreisbogen / Accordion-Zeile)

[ ] Scroll-Fallback: wenn Items nicht reinpassen → scroll (wichtig für Mobile/Smartphone)

[ ] Icon-Hintergründe: immer transparent (kein Hintergrund-Rect)

[ ] i18n: alle neuen Texte in navigation.ftl
[ ] cargo fmt + clippy + test grün
```

## G1.3 — Navigation: Komponenten (fs-components)

```
Design Pattern: Composite (MenuTree) + FsView (Domain → Widget)

[ ] CornerMenu struct + FsView impl
      field: corner, items, config (Größe, Stil)
[ ] SideMenu struct + FsView impl
      field: side, items, config
[ ] SidebarPanel struct + FsView impl
      Overlay-Sidebar: kommt über den Inhalt (Bildschirm verschiebt sich NICHT)
      Gleiche Items wie SideMenu, anderes Aussehen
      Einstellung: "Menü-Stil: Rund | Sidebar" in Settings > Ansicht

[ ] Alle drei mit scroll-Fallback (kleine Screens / Mobile)
[ ] i18n: Komponenten-Beschriftungen in navigation.ftl
[ ] cargo fmt + clippy + test grün
```

## G1.4 — Program-Modell-Erweiterungen (fs-inventory + fs-store)

```
Design Pattern: Decorator (caption/label erweitert bestehende InstalledResource)
               Composite (ProgramGroup = 1 Parent + N Instanzen)

[ ] caption: Option<String> in InstalledResource (fs-inventory)
      → Anzeigename, unabhängig vom technischen Namen
      → Notwendig wenn mehrere Versionen/Instanzen des gleichen Programms laufen

[ ] caption: Option<String> in PackageData (fs-store)

[ ] ProgramGroup struct (fs-inventory):
      parent_id: ResourceId
      instances: Vec<InstalledResource>
      group_icon: IconRef
      → Ein Parent-Icon, N Instanzen auswählbar (z.B. wiki.a, wiki.b, wiki.c)

[ ] CompositeIcon in Store-Paket-Deklaration:
      primary_icon = "icon.svg"    (Stamm-Programm)
      secondary_icon = "badge.svg" (Instanz — wird leicht überlappend angezeigt)
      overlap_factor = 0.3
      → TOML-konfigurierbar in package.toml

[ ] ProgramViewProvider trait impl für alle bestehenden Programme:
      Kanidm: Info + Manual + SettingsContainer
      Stalwart: Info + Manual + SettingsContainer
      Tuwunel: Info + Manual + SettingsContainer
      Zentinel: Info + Manual + SettingsContainer
      Forgejo: Info + Manual + SettingsContainer
      Outline/Wiki.js: Info + Manual + SettingsContainer
      Desktop: SettingsConfig + (kein Start, kein Container)

[ ] Info-View: Berechtigungscheck für kritische Aktionen (kill, restart)
      Eigener Rechner: alle Aktionen
      Server: nur mit expliziter Berechtigung (fs-auth Rights-Kaskade)

[ ] i18n: inventory.ftl ergänzen (caption, group, views)
[ ] cargo fmt + clippy + test grün
[ ] Doku: konzepte/program-views.md
```

## G1.5 — Desktop-Shell-Refactor (fs-gui-workspace)

```
Design Pattern: Facade (DesktopShell) + State Machine (MenuState) + Observer (CapabilityObserver)

Bestehende Sidebars werden durch Corner Menus ersetzt.
wallpaper.rs ist bereits vorhanden → einbinden.

[ ] Default-Layout mit Corner Menus:
      Oben links:    Task-Menü (alle installierten/laufenden Programme)
      Unten links:   Settings (SettingsConfig für Desktop)
      Oben rechts:   Help (General Help + Focus Help)
      Unten rechts:  AI (nur wenn ActivityEngine-Capability "ai.chat" vorhanden)

[ ] Titlebar-Erweiterung:
      Standard: Icon | Titel (zentriert) | [−][□][×]
      NEU:      Icon | Titel (zentriert) | [View-Buttons] | [Tiling-Toggle] | [−][□][×]
      View-Buttons: wechseln zwischen ProgramViews (Start/Info/Manual/Settings/...)
      Tiling-Toggle: ordnet offene Fenster automatisch an

[ ] Fullscreen als Standard: Desktop startet immer im Fullscreen-Modus

[ ] Hintergrund (wallpaper.rs einbinden):
      Einstellbar in Settings > Ansicht: Farbe oder Bild
      Standard: einfarbig (Desktop-Primärfarbe, dunkel)

[ ] Icon-Hintergründe: transparent (kein weißes/graues Hintergrund-Rect)

[ ] Einstellungen (Settings > Ansicht):
      Icon-Größe: Schieberegler (Standardgröße 32px, Range 16–64px)
      Menü-Stil: Rund (Corner/Side Menus) | Sidebar (SidebarPanel)
      Schriftart: kommt später (G2)

[ ] CapabilityObserver: prüft ob "ai.chat" im fs-registry läuft
      ja → unten rechts AI-Corner-Menu einblenden
      nein → unten rechts leer

[ ] i18n: desktop.ftl (alle neuen Texte)
[ ] cargo fmt + clippy + test grün
[ ] Doku: programme/fs-desktop.md überarbeiten
```

## G1.6 — Activity Hub (fs-components + eigene Repos)

```
Design Pattern: Registry (ActivityHub) + Strategy (ActivityEngine pro Programm)
               Composite (Activity enthält Sub-Activities)

Kernkonzept: Jede Activity IST ein Program-Angebot.
Ein Programm kann einen ActivityEngine-Adapter mitbringen und wird damit zur Activity.
Activity Hub fragt nur den Trait — egal ob Container, eingebettetes Modul oder WASM-Plugin.

[ ] ActivityEngine trait (fs-render):
      activity_id() -> &str
      activity_name_key() -> &str
      supported_actions() -> Vec<ActivityAction>
        ActivityAction: New | Open | Recent | Browse | Custom(String)
      default_view() -> ProgramView
      category() -> ActivityCategory
        ActivityCategory: Document | Communication | Media | Data | Code | System | Custom(String)

[ ] Activity struct (fs-components):
      id, icon: CompositeIcon, name_key, category
      engine: Arc<dyn ActivityEngine>
      sub_activities: Vec<Activity>
      → Nesting: "Office" enthält "Write", "Spreadsheet", "Presentation"
      → "Office" selbst ist eine Activity (eigenes Icon, eigene Aktionen)

[ ] ActivityHub struct (fs-components):
      activities: Vec<Activity>   (per Store ladbar, per fs-inventory gefiltert)
      Registry-Methoden: register(activity), by_category(), search(query)
      Nur installierte Aktivitäten werden angezeigt (fs-inventory gRPC)

[ ] Activity-Views (was in der Activity angezeigt wird):
      "Neu"              — neues Dokument/Datei/Termin/... erstellen
      "Zuletzt bearbeitet" — letzte N Elemente (aus fs-session oder Programm)
      "Programme"        — 2. Ebene: welches Programm für diese Activity nutzen?
      Standard-Programm pro Activity konfigurierbar
      Start-View pro Activity konfigurierbar (Einstellung: womit öffne ich die Activity)

[ ] Activity als eigenständiges Programm (optional):
      Eigener Container + Adapter (Adapter implementiert ActivityEngine)
      Eigene GUI optional (via FsView-Trait)
      Im Store als program-Paket mit ActivityEngine-Capability
      Kann auch eingebettet bleiben (kein eigener Container nötig)

[ ] ActivityHub als Corner-Menu-Entry oder Widget
      Einstellbar: als Corner Menu | als Desktop-Widget | beides

[ ] i18n: activity-hub.ftl (en + de)
[ ] cargo fmt + clippy + test grün
[ ] Doku: konzepte/activity-hub.md
```

## G1.7 — Content-Komponenten

```
Design Pattern: Strategy (HelpSource) + Observer (FocusObserver) + Facade (SettingsHub)

[ ] InventoryComponent (fs-render Standard-Komponente erweitern):
      Zeigt alle installierten Programme (fs-inventory gRPC)
      Gruppen-Darstellung: ProgramGroup mit aufklappbaren Instanzen
      Composite-Icons für Instanzen
      Slot: fill (Sidebar oder Main)

[ ] GeneralHelpComponent:
      Was kann man auf dieser Maske tun?
      Welche Aktionen sind verfügbar?
      Kontext-sensitiv per HelpSystem (fs-help)
      Jedes Programm pflegt eigene Hilfetexte (.ftl in fs-i18n)

[ ] FocusHelpComponent:
      Erklärt das Element mit aktuellem Fokus
      Beispiel: Email-Input → zeigt gültige Zeichen, Format-Beispiel
      Jedes Input-Element kann Hilfe-Metadaten tragen (help_key: Option<String>)
      Wird auch in TUI angezeigt (gleiche Traits)

[ ] SettingsConfigComponent (fs-settings):
      Multi-Section, passt sich Fenster/Programm an
      Sektionen: Ansicht, Schrift (später), Sprache, Hintergrund, Tastaturkürzel, ...
      Desktop: zeigt Desktop-spezifische Einstellungen
      Andere Programme: zeigen ihre programmspezifischen Einstellungen
      Sofort-Änderung (kein Neustart)

[ ] SettingsContainerComponent (fs-container-app):
      Pod-YAML-Konfig für Container-Programme
      Änderung erfordert Restart (Hinweis im UI)
      Instanz kopieren: Neue Instanz mit gleicher Konfig + anderem Namen
      Dann: Instanz starten, Instanz stoppen, Instanz löschen
      Berechtigungscheck: nicht jeder darf Container konfigurieren

[ ] SearchComponent (fs-desktop, Phase 6.2 erweitern):
      Input-Feld (Schnellsuche über alles)
      Erweiterter Bereich (aufklappbar):
        Nach Tags suchen
        In bestimmtem Programm suchen
        In Programm-Gruppe suchen
        Programm-übergreifend suchen
      Resultat: LensItems gruppiert nach Quelle
      i18n: search.ftl ergänzen

[ ] AiComponent (fs-ai — NICHT in fs-desktop):
      Textbox für KI-Chat
      Nur eingeblendet wenn "ai.chat" Capability vorhanden (fs-registry)
      Gehört in fs-ai Repo, nicht in allgemeine Komponenten
      Wird als Corner-Menu-Entry eingebunden (unten rechts)

[ ] i18n: alle Komponenten in passende .ftl-Dateien
[ ] cargo fmt + clippy + test grün
```

## G1.8 — Offene Items (konsolidiert aus G0)

```
Settings-Seiten (fs-settings):
[ ] Appearance-Settings: Theme-Wahl via fs-theme
[ ] Language-Settings: Sprache global + per App (fs-manager-language)
[ ] Service Roles Settings: Dienst-Rollen konfigurieren
[ ] Accounts-Settings: IAM via Kanidm (OIDC-Login-Flow)
[ ] Shortcuts-Settings: Tastaturkürzel-Editor
[ ] Packages-Settings: Store-View direkt in Settings

Desktop-Shell (fs-gui-workspace):
[ ] Hot-Reload Layout via inotify (Phase 3 HotReloadWatcher einbinden)
[ ] IcedLayoutInterpreter vollständig in Shell-View einbinden
[ ] AiHelpSource: echte gRPC-Anbindung an fs-ai

Store (fs-store + Store/):
[ ] Alle catalog.toml vollständig ergänzen:
    kanidm, stalwart, tuwunel, zentinel, telegram, outline, wikijs,
    fs-desktop, fs-init, fs-store, fs-auth, fs-managers,
    Render-Engines, DB-Engines (als Artifacts)
[ ] Screenshots-System: Store zeigt Bilder im Detail-Panel
[ ] Doku: technik/store-catalog-spec.md anlegen
[ ] BrowseMode: "Verfügbar"-Tab (alle Store-Pakete, auch nicht installierte)
[ ] "Installieren"-Button nur aktiv wenn fs-store Service läuft

Datenbank (fs-db):
[ ] CrudRepo → Repository<T> migrieren
[ ] Filter<T> → SQL in Adapter-Repos übersetzen

Bots (fs-bots):
[ ] bot-db/src/lib.rs aufteilen: conversation.rs + user.rs + ...
[ ] DB nur über fs-db DbEngine-Trait

Search (fs-lenses + fs-search):
[ ] BusSearchStrategy: echte fs-bus Integration (aktuell Demo-Impl)
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

## G1.9 — UX-Extras (nach G1.5)

```
Design Pattern: Decorator (Badges) + Observer (FocusObserver) + Command (QuickSwitch)

[ ] Status-Badges auf Icons:
      ungelesen (Zahl), Update verfügbar (Punkt), Fehler (Ausrufezeichen), laufend (Kreis)
      Implementierung: Badge-Overlay auf CompositeIcon

[ ] Thumbnail-Preview bei Hover auf laufende Programme:
      Mini-Vorschau des Programmfensters im Task-Menü

[ ] Focus Mode:
      Alles ausblenden — nur ein Fenster sichtbar
      Toggle im Tiling-Toggle-Button der Titlebar

[ ] Quick Switch Overlay (Alt+Tab-Ersatz):
      Tastaturkürzel öffnet Overlay mit allen offenen Fenstern
      Thumbnail-Previews, tastaturnavigierbar
      FSView-Trait → funktioniert in iced + bevy (nicht TUI)

[ ] Notification Center:
      Alle Toasts aggregiert abrufbar (Corner Menu oder eigener Bereich)
      Ungelesene werden markiert

[ ] Workspace Profiles:
      Layout-Presets speichern (z.B. "Coding", "Writing", "Admin")
      In Settings > Ansicht verwaltbar

[ ] Touch Gestures (Mobile / Touchscreen):
      Swipe von Ecke = Corner Menu öffnen
      Swipe von Kante = Side Menu öffnen
      Pinch-to-zoom auf Desktop-Hintergrund

[ ] Pinboard Widget:
      Fixierbare Notizen/Links auf dem Desktop-Hintergrund
      Als Activity oder als eigenständiges Widget

[ ] Auto Dark/Light:
      Thema wechselt automatisch je nach Tageszeit
      Konfigurierbar in Settings > Ansicht (Zeitfenster)

[ ] Breadcrumb in Titlebar:
      Für Programme mit hierarchischer Navigation (Browser, Store, Docs)
      Bereits header.rs vorhanden → einbinden

[ ] Global Clipboard History:
      Alle kopierten Inhalte abrufbar (Text, Bilder, Links)
      Über Search-Komponente abrufbar (Kategorie "Clipboard")
      Lokaler Store (verschlüsselt), kein Cloud-Sync

[ ] Window Snap:
      Automatische Anordnung bei mehreren offenen Fenstern
      Ergänzt den Tiling-Toggle (manuell vs automatisch)
      Snap-Zonen: 2er/3er/4er Raster, konfigurierbar
      Tiling-Toggle: Manuell | Auto-Snap | Focus Mode
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
G1.9  UX-Extras                           ← nach G1.5

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
