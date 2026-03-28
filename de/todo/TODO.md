# FreeSynergy — Build Plan

[← Zurück zum Index](../INDEX.md)

---

## TODO-Regeln (für Claude-Sessions)

- **TODO.md immer vollständig lesen** zu Beginn jeder Session — auch wenn es Token kostet.
  Nur so gibt es einen vollständigen Überblick ohne Missverständnisse.
- **Erledigte Items löschen** — kein [x] behalten. Löschen spart Token in zukünftigen Sessions.
  Abgeschlossene Phasen werden auf eine Zeile komprimiert.
- **Vor jeder Aktion:** Was braucht dieses Modul? Store als Single Source of Truth prüfen.
- **Cleanen:** gehe bitte alle repos durch und räume auf. schau, ob alles OK ist. sei es im Repository, sei es, ob sie vollständig sind und ob man sie compilieren kann … schau nach, das gute design patterns genommen wurden und die Dateien nicht zu groß sind …

---

## Qualitäts-Regeln (für ALLE Repos, IMMER)

Diese Regeln gelten ohne Ausnahme. Kein Commit ohne grüne Gates.

**Reihenfolge bei jedem Programm / Modul:**
```
1. Design Pattern festlegen (Traits, Objekt-Hierarchie, Strategy/Observer/etc.)
2. Structs + Traits definieren — noch kein Impl-Code
3. cargo check — Fehler beheben
4. Impl schreiben (OOP: Objekte statt Daten, Traits statt match-Blöcke)
5. cargo clippy --all-targets -- -D warnings
6. cargo fmt --check
7. Unit Tests schreiben (mind. 1 Test pro öffentlichem Modul)
8. cargo test
9. Erst dann: commit + push
```

**Jedes Repo muss haben:**
- `#![deny(clippy::all, clippy::pedantic)]` in jeder `lib.rs` und `main.rs`
- `#![deny(warnings)]`
- `rustfmt.toml`
- `deny.toml` (cargo-deny)
- `LICENSE` (MIT)
- `README.md` (Zweck, Build-Anleitung, Kurz-Architektur)
- `assets/icon.svg` (Platzhalter-Icon — auch wenn noch nicht final)
- `CLAUDE.md` (diese Qualitäts-Regeln, für Claude-Sessions)

**Dokumentation:**
- Betroffene Doku-Seite ZUERST komplett lesen
- Dann NEU schreiben — nicht anhängen
- Widersprüche aktiv suchen und korrigieren
- Immer: commit + push zu `fs-documentation`

---

## Phase A: Dokumentation aktualisieren ✅ (2026-03-26)

## Phase B: Store bereinigen ✅ (2026-03-26)

---

## Phase C: Repositories anlegen und vorbereiten ✅ (2026-03-26)

---

## Phase D: fs-libs schrumpfen

> Ziel: fs-libs enthält am Ende nur wenige echte, universelle Primitives.
> Alle anderen Crates bekommen eigene Repos oder wandern in ihr Programm-Repo.

> **Grundregel — wie Linux:**
> Jedes Modul ist klein, fokussiert, eigenständig versionierbar.
> Zusammenlegen nur wenn es einen echten fachlichen Grund gibt — nicht umgekehrt.
> Lieber ein Repo mehr. Trennen ist die Regel, Mergen die Ausnahme.
> Separates Repo = separate Version = unabhängige Updates = kein Aufblähen.
> Vorher immer prüfen: Wer nutzt dieses Modul? Gibt es eine API im Store?

**Was bleibt in fs-libs (echte universelle Primitives — winzig, stabil, alle brauchen sie):**
```
fs-types    — FsValue, FsUrl, SemVer, LanguageCode, FsPort, FsTag (Basis-Typen)
fs-error    — Basis-Fehler-Infrastruktur (alle Programme brauchen Fehlertypen)
fs-crypto   — Verschlüsselung (alle Services brauchen Crypto)
fs-health   — Health-Check-Trait (alle Services implementieren ihn)
fs-core     — FsManager-Trait, ManagerRegistry, ManagerStore (zero deps; 5 Manager + Desktop nutzen es)
```

---

**Bereits erledigt ✅:** fs-bus | fs-config | fs-db | fs-auth | fs-federation | fs-i18n | fs-theme | fs-ui | fs-components | fs-render | fs-llm | fs-channel | fs-bot (→ fs-bots) | fs-sysinfo (→ fs-info) | fs-pkg+fs-plugin-sdk+fs-plugin-runtime (→ fs-packages) | fs-container → migriert | fs-help (→ eigenes Repo) | fs-core (bleibt in fs-libs als 5. Primitiv)

---

**Strings aufräumen — Migration zu fs-i18n (parallel zu Phase E):**
```
D17.[ ] Code-Audit: alle hardcodierten Strings finden
        grep nach: println!, format!, eprintln!, direkte String-Literals in UI
        Gilt für ALLE Repos, nicht nur fs-libs

D18.[ ] common.ftl anlegen in fs-i18n/locales/de/ und /en/
        Inhalt: alle wiederverwendbaren Fehlermeldungen
        error-not-found, error-permission-denied, error-invalid-input,
        error-connection-failed, error-parse-failed, ...
        → Einmal definiert, überall referenziert

D19.[ ] Pro Programm: {programm}.ftl in fs-i18n/locales/{lang}/
        Reihenfolge: parallel zu Phase E — je Programm beim Sauber-Machen
        Regel: KEIN roher String in Code — immer nur i18n-Keys
```

---

**Offen (blockiert):**
```
D24.[ ] fs-bridge-sdk → löschen
        Erst löschen wenn fs-registry läuft und alle Bridges migriert sind
```

---

**D25 ✅:** fs-libs committed + pushed — enthält jetzt nur fs-types, fs-error, fs-crypto, fs-health, fs-core

---

## Phase E: Programme — Einzeln sauber machen

> Jedes Repo wird komplett durchgearbeitet.
> Reihenfolge: Klein → Groß. Einfach → Komplex.
> Vorlage für alle: fs-browser (erstes sauberes End-to-End-Beispiel).

**Jedes Repo bekommt diese Checkliste (Kurzform: "Repo-Gate"):**
```
[ ] CLAUDE.md mit Qualitäts-Regeln
[ ] rustfmt.toml vorhanden
[ ] deny.toml vorhanden
[ ] LICENSE (MIT)
[ ] README.md (Zweck, Build, Architektur)
[ ] assets/icon.svg (Platzhalter)
[ ] #![deny(clippy::all, clippy::pedantic, warnings)] in allen lib.rs/main.rs
[ ] cargo clippy --all-targets -- -D warnings: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: alle Tests grün
[ ] cargo build --release: funktioniert
[ ] Doku-Seite neu geschrieben
[ ] commit + push
```

**Reihenfolge:**

**Erledigt ✅:** fs-config, fs-bus, fs-db, fs-inventory, fs-session, fs-registry, fs-init, fs-icons, fs-browser, fs-store, fs-lenses, fs-tasks, fs-bots, fs-managers, fs-container-app, fs-ai, fs-node, fs-desktop

**Phase E abgeschlossen ✅**

---

## Phase F: Integration

> Ziel: Die Programme reden miteinander.
> Erst wenn alle Repos aus Phase E ihren Repo-Gate bestanden haben.

```
F1. ✅ fs-store ↔ Store/ Kompatibilitäts-Test
        97 Pakete geladen (18 apps + 7 managers + 10 containers + ...),
        [[binaries]] + [distribution] + [interfaces] geparst,
        managers als eigener Namespace (war fälschlich als App referenziert)

F2. ✅ Manager → fs-inventory
        ThemeManager::install_from_store(inventory, id, version) implementiert
        2 Integration-Tests grün: recorded + idempotent upsert

F3. ✅ fs-bus verdrahten
        InventoryBusHandler: installer.# → upsert_resource / uninstall
        RegistryBusHandler: service.# → register / deregister
        4 Integration-Tests grün (fs-bus/tests/bus_wiring.rs)

F4. ✅ fs-session in Desktop einbauen
        SessionTracker in Desktop: open/minimize/restore/close → SessionStore (fire-and-forget)
        session_tracker Signal via use_effect + spawn initialisiert

F5. ✅ fs-registry → Adapter-Lookup
        Registry::endpoint_for_capability(capability) → Option<String>
        4 Tests: returns up service, skips down, returns none, prefers up over down

F6. ✅ End-to-End Test: Paket installieren
        StoreReader: distribution URLs jetzt in PackageRelease gespeichert (fs-store)
        4 Tests in fs-bus/tests/e2e_install.rs:
          catalog_manager_has_linux_download_url → URL-Template mit {version} vorhanden
          install_event_recorded_in_inventory → bus event → inventory entry
          service_start_event_registered_in_registry → bus event → registry entry
          full_install_chain → kompletter Flow: catalog → install → start → endpoint_for_capability
```

---

## Nächste Gespräche

> Diese Themen müssen besprochen werden bevor sie umgesetzt werden können.
> Kein Code ohne Gespräch und Entscheidung.

```
G1. fs-auth Design + fs-node Architektur
    ENTSCHEIDUNG (2026-03-27): Protokoll-Traits-Ansatz — modular, Bus-kompatibel.
    fs-auth definiert 4 separate Protokoll-Traits. Kanidm implementiert alle 4.
    Jedes zukünftige Auth-Backend implementiert nur die Protokolle die es unterstützt.

    AUTH — Umsetzung:
    G1.1 ✅ fs-auth: 4 Protokoll-Traits + AuthCapabilities (2026-03-27)
    G1.2 ✅ KanidmBackend: echte reqwest-Impl für alle 4 Traits — PAM (3-step /v1/auth), SCIM (/v1/person), OAuth2 (/oauth2/*), SSO (/v1/auth/valid) (2026-03-27)
    G1.3 ✅ fs-auth: Login-UI + CLI (2026-03-27)
              LoginView (fs-render Widgets: username/password/submit-btn)
              CLI binary `fs-auth`: login / logout / status / provision
              Alle Operationen über Traits — nie Kanidm direkt
    G1.4 ✅ Bus-Integration (2026-03-27)
              AuthBusPublisher: publish(AuthEvent) → fs-bus
              auth.login / auth.logout / auth.provision
              Feature-gated: kanidm / bus / ui / cli

    NODE — Orchestrierungs-Schichten:
    G1.5 ✅ fs-node: Grundstruktur (2026-03-28)
              NodeLayer-Trait: start/stop/name Lifecycle
              AuthGateway    → fs-auth (OAuthProvider + PamProvider + SsoProvider)
              S3Provider     → fs-s3 (opendal)
              ServiceProxy   → fs-registry (Capability-Lookup)
              FederationGate → Trait + disabled placeholder (bis Phase P)
              NodeServer     → axum HTTP API; `fsn node serve`
    G1.6 ✅ fs-node: Einladungs-System (2026-03-28)
              InviteToken: fsn1.… HMAC-signierter Token (node_id|address|expires|nonce)
              InviteBundle: age-verschlüsseltes TOML-Paket (passphrase-basiert)
              PortPool: dedizierter Port pro Einladung (konfigurierbarer Range)
              InviteSystem: create / verify / accept
              CLI: `fsn node invite create` / `fsn node invite accept`

    HINWEIS S3: MinIO wird NICHT verwendet.
    fs-s3 nutzt `s3s` + `s3s-fs` (MIT) als eingebetteten S3-Server.
    `opendal` (Apache-2.0) übernimmt austauschbare Replikations-Backends
    (Hetzner Storagebox, S3-remote, SFTP). Kein Lizenzproblem.

G2. Desktop Rendering-Architektur
    ENTSCHEIDUNG (2026-03-27): Abstraktions-API zuerst, dann austauschbare Engines.
    Kein Code in fs-desktop/fs-apps der direkt iced, bevy oder Servo importiert.
    Dioxus wird vollständig entfernt. fs-render wird neu aufgebaut.

    RENDER-LAYER (GUI):
    G2.1 ✅ fs-render: RenderEngine, FsWidget, FsWindow, FsTheme, FsEvent, AnimationSet — 19 Tests grün (2026-03-27)
    G2.2 ✅ fs-gui-engine-iced — IcedEngine/IcedWindow/IcedWidget/IcedTheme, iced 0.13, 16 Tests grün (2026-03-27)
    G2.3 ✅ fs-gui-engine-bevy (2026-03-28)
              BevyEngine/BevyWindow/BevyWidget/BevyTheme — alle fs-render Traits
              FsRenderPlugin: Bevy Plugin, init_resource<WorkspaceScene>, tick_animated_backgrounds
              WorkspaceScene (Resource) + WorkspaceSceneBuilder (Builder Pattern)
              WorkspaceNode (Builder: with_icon/at_position/active), WorkspaceLayout (Grid/Circular/Freeform)
              AnimatedBackground (Component): Particles/Gradient/WaveField/Static + tick/pause/resume
              25 Unit-Tests + 1 Doc-Test, clippy clean

    ANIMATIONS-SYSTEM (Teil von fs-render, Store-erweiterbar):
    G2.4 ✅ AnimationSet als Store-Paket (2026-03-28)
              AppAction enum: 16 Standard-Actions + Custom(String)
              AnimationActionMap: AppAction → AnimationActionEntry (Registry Pattern)
              DefaultAnimationSet: 7 Animations + 16 Action-Mappings (slide/fade/scale/rotate)
              AnimationRegistry: load_set / activate / resolve / effective_speed
              ActionAnimationOverride: per-Action speed + disable
              8 Tests grün, WASM-Erweiterung via fs-plugin-sdk (später)

    BROWSER-LAYER:
    G2.5 ✅ fs-web-engine — WebEngine/WebView/WebPlugin Traits, WebCapabilities, NavigationHistory,
              JsValue, WebUrl, WebError, StubWebEngine/View — 16 Tests grün (2026-03-28)
    G2.6 ✅ fs-web-engine-servo — ServoWebEngine/View/PluginRegistry, Request-Interception + Redirect,
              feature-gated servo Integration — 27 Tests grün (2026-03-28)
    G2.7 ✅ fs-browser anpassen (2026-03-28)
              Migriert zu iced via fs-gui-engine-iced (Commit: "migrate browser from Dioxus to iced")
              Feature-Flag: iced-gui (default) + servo (future)

    INTEGRATION:
    G2.8 [ ] fs-desktop anpassen — ⚠️ COMPILE-FEHLER (blockiert alles in fs-desktop!)
              Problem: workspace.dependencies hat kein dioxus mehr, ABER:
                - fs-settings/Cargo.toml: noch `dioxus = { workspace = true }`
                - fs-profile/Cargo.toml: noch `dioxus = { workspace = true }`
                - fs-showcase/Cargo.toml: noch `dioxus = { workspace = true }`
                - Alle .rs in fs-gui-workspace + fs-settings + fs-profile + fs-showcase: `use dioxus::prelude::*`
              Aktion: Dioxus vollständig entfernen, auf fs-render/iced migrieren
              Engine-Auswahl via Feature-Flag (default: iced)
    G2.9 [ ] fs-settings/fs-desktop Dioxus-Inhalt nach G2.8 migrieren
              Daten-Structs in fs-settings sind fertig (KDE-Konfig: Fenster, Click, Animationen, Theme,
              Icons, Cursor, Shortcuts, Workspace) — Commit existiert "feat(fs-settings): G2.9"
              ABER: alles noch in Dioxus-Code — muss auf iced/fs-render umgeschrieben werden

G3. Bus API Namespaces (Vertrags-Design)
    - Welche Bus-Message-Namespaces brauchen wir?
    - Payload-Format (Typen aus fs-types)
    - Wer darf was publizieren? Wer darf was subscriben?

G4. CI/CD Workflow Template
    - GitHub Actions: ein Template für Native-Apps, eines für Libs
    - Wann wird gebaut? (bei Tag, bei Push auf main?)
    - Wie kommen Binaries in GitHub Releases?
    - Wie wird der Store/-Katalog automatisch aktualisiert?

G5. fs-managers UI-Design (pro Manager)
    - Gemeinsames Layout-Muster für alle Manager
    - Was zeigt jeder Manager? (Liste + Konfig + Status)
    - Besonders: Bot-Manager (Messenger-Verbindung) und AI-Manager (LLM-Auswahl)

G6. Forks — Build-Strategie
    - Kanidm, Tuwunel, Stalwart, Mistral, Zentinel
    - Nur kompilieren, nicht ändern
    - Wie werden sie als App-Pakete in den Store gebracht?
    - GitHub Actions: automatischer Sync mit Upstream

G7. fs-db Design
    - Was ist die DB-Abstraktion genau?
    - Welche Programme brauchen sie? (alle die SQLite nutzen)
    - sea-orm als Basis — Schema-Migration-Strategie

G8. Daemon vs. Bus-Subscriber vs. Library — wann brauchen wir was?
    Hintergrund: fs-info, fs-inventory, fs-session, fs-registry sind als "Daemons"
    beschrieben — aber wenn wir Event-Handling und Queues über fs-bus haben, brauchen
    wir dann überhaupt eigene Prozesse?

    Optionen:
      Library:        direkte Einbindung, kein eigener Prozess, kein Netzwerk-Overhead
                      Pro: einfach, schnell | Con: jeder Nutzer trägt den Code selbst
      Bus-Subscriber: Service läuft, hört auf Bus-Messages, antwortet auf Anfragen
                      Pro: entkoppelt, zentrale Daten | Con: Bus muss laufen
      Daemon:         eigenständiger Prozess mit eigenem API (HTTP, gRPC, Socket)
                      Pro: unabhängig vom Bus | Con: ein Prozess mehr

    Zu entscheiden: Ist fs-bus zuverlässig genug als einzigen Kommunikationsweg?
    Falls ja → Bus-Subscriber reicht für fs-info, fs-inventory etc., kein Daemon nötig
    Falls nein → Daemon mit Bus-Integration als Fallback

    Betrifft: fs-info (C13/D14), fs-inventory, fs-session, fs-registry
```

---

## Phase H: Repo-Aufräumen (2026-03-28 — Audit-Ergebnis)

> Befunde aus vollständigem Repo-Scan aller 30+ Repos.
> Reihenfolge: Kritisch → Architektur → Große Dateien → Docs

```
H1. [ ] ⚠️ KRITISCH: fs-desktop kompiliert nicht
        Ursache: G2.8 halb-fertig — dioxus aus workspace.dependencies entfernt,
        aber fs-settings/fs-profile/fs-showcase Cargo.tomls + alle .rs-Dateien
        noch auf Dioxus. Blockiert jede Arbeit an fs-desktop.
        → Vor H1 muss G2.8 erledigt werden.

H2. [ ] Orphan-Repo fs-core (standalone) aufräumen
        /home/kal/Server/fs-core/ ist ein Überbleibsel der D22/D23-Migration.
        fs-core wurde in D25 zurück in fs-libs geholt (Commit: "restore fs-core into fs-libs").
        Standalone Repo hat nur 1 Commit, identischen Inhalt, wird nirgends referenziert.
        Aktion: Repo archivieren oder löschen.

H3. [ ] MEMORY.md: fs-bus, fs-config, fs-db als eigene Repos eintragen
        Alle drei existieren, kompilieren, sind aktiv als Abhängigkeiten genutzt — fehlen aber
        in der Repo-Tabelle in MEMORY.md.
        fs-bus:    git@github.com:FreeSynergy/fs-bus.git
        fs-config: git@github.com:FreeSynergy/fs-config.git
        fs-db:     git@github.com:FreeSynergy/fs-db.git

H4. [ ] Große Datei aufteilen: fs-packages/fs-pkg/src/manifest.rs (1394 Zeilen)
        → Aufteilen in: manifest/fields.rs, manifest/validation.rs, manifest/builder.rs o.ä.

H5. [ ] Große Datei aufteilen: fs-components/src/nav.rs (1204 Zeilen)
        → Aufteilen in kleinere Komponenten-Module

H6. [ ] Große Datei aufteilen: fs-packages/fs-pkg/src/setup_step.rs (886 Zeilen)
        → Aufteilen nach Step-Typen

H7. [ ] Große Datei aufteilen: fs-packages/fs-pkg/src/installer.rs (706 Zeilen)
        → Aufteilen nach Installer-Phasen

H8. [ ] Große Datei aufteilen: fs-bots/bot-db/src/lib.rs (735 Zeilen)
        → Aufteilen nach DB-Entitäten/Operationen

H9. [ ] Große Dateien in fs-apps/crates/fs-managers (nach G2.8):
        language_panel.rs: 1060 Zeilen
        cursor_panel.rs:    808 Zeilen
        manager_view.rs:    792 Zeilen
        install_wizard.rs:  606 Zeilen (fs-store-app)
        → Panels nach Sub-Abschnitten aufteilen

H10.[ ] Große Dateien in fs-desktop (nach G2.8/G2.9):
        fs-gui-workspace/src/desktop.rs:              1362 Zeilen
        fs-settings/src/desktop_settings.rs:          1172 Zeilen
        fs-settings/src/language.rs:                  1150 Zeilen
        → Nach G2.8-Migration in kleinere Module aufteilen

H11.[ ] fs-ui description + README updaten
        Cargo.toml noch: "Dioxus UI component library" — nach G2-Entscheidung veraltet.
        fs-ui gehört zu fs-render/iced Welt, nicht mehr Dioxus.

H12.[ ] fs-node: kein Cargo.toml im Root
        fs-node hat cli/, validator/, migration/ mit eigenen Cargo.tomls aber kein
        Workspace-Cargo.toml im Root. Prüfen ob das gewollt ist oder ob ein Workspace fehlt.

H13.[ ] D17-D19: i18n-Audit noch offen
        Code-Audit auf hardcodierte Strings, common.ftl + programm.ftl anlegen.
        Gilt für ALLE Repos. (Parallel zu anderen Phasen möglich)
```

---

## Archiv / Ideen-Pool

> Gute Ideen die noch nicht dran sind.
> Nicht löschen — wenn das Thema aktuell wird, in die aktiven Phasen verschieben.
> Wenn ein Thema sich verändert hat und die Idee nicht mehr passt → streichen.

### Search (Phase M — nach Phase F)
```
M1. [ ] Search-View (Suchfeld, gruppierte Ergebnisse, Preview + Link)
M2. [ ] Service-Suche (Ebene 1 — lokal)
M3. [ ] Host-Suche (Ebene 2, Bus-aggregiert)
M4. [ ] Föderale Suche (Ebene 3-4, nur mit search-Recht)
```

### Federation + Node-Netzwerk (Phase P — nach G1)
```
P1. [ ] Invite-System (Token, verschlüsselte TOML, Port pro Einladung)
P2. [ ] Federation-Grundstruktur (Domain-Pflicht, Auth-Broker)
P3. [ ] Rechte-Kaskade (read/write/execute/search, Audit-Log)
P4. [ ] Föderaler Bus (Node-zu-Node Kommunikation)
```

### Mail (Phase R — nach Federation)
```
Entscheidung: Stalwart unverändert als App-Paket (kein Fork, AGPL).
Multi-Tenancy = dezentral: jeder Node, seine eigene Stalwart-Instanz.

R1. [ ] Stalwart als App-Paket (Store-Eintrag, Binary aus Fork)
R2. [ ] IAM-Integration (Kanidm via OIDC/LDAP)
R3. [ ] Adapter implementieren (smtp, imap Capabilities in Registry)
R4. [ ] Domain-Konfiguration pro Node
```

### Kontakte & Kalender (Phase S — nach Mail)
```
S1. [ ] Rustical evaluieren (CalDAV + CardDAV, Lizenz prüfen)
S2. [ ] Kontakte-Backend (vCard 4.0, SQLite + S3)
S3. [ ] Kalender-Backend (iCal/CalDAV, icalendar Crate)
S4. [ ] IAM-Integration (Kanidm → automatisch Kalender + Adressbuch)
S5. [ ] App-Pakete: contacts-server, calendar-server
```

### Infrastruktur-Apps (Phase T — nach Federation)
```
T1. [ ] Vaultwarden (Passwort-Manager, Bitwarden-kompatibel, AGPL)
T2. [ ] Ntfy / UnifiedPush (Push-Benachrichtigungen, ohne Google/Apple)
T3. [ ] Element Call + coturn (Video-Calls via Tuwunel)
T4. [ ] WireGuard (Node-zu-Node VPN, nur wenn Federation läuft)
T5. [ ] Hickory DNS (internes DNS, nur nach Federation)
```

### Polish (Phase Q — nach Phase F)
```
Q1. [ ] Action Registry + konfigurierbare Shortcuts
Q2. [ ] Hilfe: Auto-generierte Shortcut-Referenz
Q3. [ ] Menü: JEDER Punkt ruft echte Aktion auf
Q4. [ ] Profil: IAM + editierbar + Account-Linking
Q5. [ ] Notification Bell
Q6. [ ] Context-Menüs
Q7. [ ] Animationen konfigurierbar
Q8. [ ] Alle Stubs / toten Code entfernen
```

### matrix-sdk State Store (Blocker: fs-db PostgreSQL)
```
N0. [ ] matrix-sdk PostgreSQL State Store implementieren
        Erst wenn fs-db auf PostgreSQL migriert ist.
        Bis dahin: In-Memory-Store in fs-channel.
        Bekannter Bug: matrix feature → recursion overflow in rustc ≥1.94 (upstream)
```

### Tasks-System (Phase O — nach Phase E14)
```
O1. [ ] Data Offers/Accepts
O2. [ ] Task Builder UI
O3. [ ] Task-Templates aus Store
```

---

## Reihenfolge (Gesamt)

```
A  Dokumentation aktualisieren          ✅ (2026-03-26)
B  Store bereinigen                     ✅ (2026-03-26)
C  Repositories anlegen                 ✅ (2026-03-26)
D  fs-libs schrumpfen                   ✅ (2026-03-26) — D17–D19 (i18n) + D24 noch offen
E  Programme einzeln sauber machen      ✅ (2026-03-28)
F  Integration                          ✅ (2026-03-28) — F1–F6 grün
G  Architektur-Gespräche                ← laufend — G2.8/G2.9 offen, G3–G8 ausstehend
H  Repo-Aufräumen (Audit-Ergebnis)      ← H1 kritisch (fs-desktop compile-fehler)
—  Archiv-Phasen                        ← wenn Zeit da ist
```
