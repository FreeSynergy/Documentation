# FreeSynergy — Build Plan

[← Zurück zum Index](../INDEX.md)

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

## Phase A: Dokumentation aktualisieren

> Ziel: Die Doku spiegelt den aktuellen Ist-Stand wider — keine Phantasie-Inhalte mehr.
> Jede Seite: erst lesen, dann neu schreiben.

```
A1. [ ] architektur/repositories.md — neu schreiben
        - Neue Repo-Struktur (eigene Repos statt Monorepo)
        - fs-libs = nur Primitives (types/error/crypto/health/i18n)
        - fs-bus, fs-db, fs-config = eigenständige Platform-Services
        - fs-managers Workspace (language/theme/icons/cursor/container/bots/ai)
        - Tabelle: Repo → Zweck → GitHub-URL → Lokal-Pfad

A2. [ ] architektur/uebersicht.md — neu schreiben
        - Schichten-Diagramm: Primitives → Platform-Services → Programme → Store
        - Inventory/Session/Registry als eigenständige Services einzeichnen
        - fs-node als zentraler Server (Auth, S3, Federation, externer Zugriff)

A3. [ ] konzepte/pakete.md — neu schreiben
        - Package-Format: [[binaries]] statt binary (mehrere Binaries pro Paket möglich)
        - capabilities + bus_messages im Katalog-TOML
        - Drei Ebenen: Store (möglich) | Inventory (installiert) | Registry (läuft)
        - Archiv-Konzept: Pakete die noch nicht bereit sind landen in packages/archive/

A4. [ ] konzepte/inventory.md — neu schreiben
        - Zwei verschiedene Inventory-Konzepte klar trennen:
          1. fs-store intern: InstallRecord (welche Version ist installiert, Pin-State)
          2. fs-inventory Service: ServiceInstance (läuft auf Port X, hat Rolle Y)
        - Wer schreibt wohin (Manager → fs-inventory, Store → InstallRecord)

A5. [ ] konzepte/adapter.md — neu schreiben (ersetzt Bridges komplett)
        - Adapter-Pattern: jeder externe Dienst implementiert einen Trait
        - Registry als Lookup: wer bietet Capability X?
        - Kein Bridge-Executor mehr — nur Trait-Impl + Registry-Eintrag

A6. [ ] technik/build-workflow.md — NEU erstellen
        - Design Pattern → OOP → cargo check → clippy → fmt → tests → commit
        - Qualitäts-Gates (was muss grün sein vor jedem Push)
        - GitHub Actions Template (wird für jeden Repo-Typ beschrieben)
        - Binary-Distribution: Repo → GitHub Release → Store-Katalog → fs-store Download

A7. [ ] INDEX.md — aktualisieren
        - Alle neuen Seiten eintragen
        - Veraltete Links korrigieren (Bridges → Adapter, Builder → Container Manager)

A8. [ ] todo/TODO.md — das ist diese Datei, sie ist bereits aktuell
```

---

## Phase B: Store bereinigen ✅ (2026-03-26)

> Ziel: Der Store enthält nur Pakete die tatsächlich existieren oder realistisch bald kommen.
> Nichts löschen — Phantasie-Pakete in `packages/archive/` verschieben.

```
B1. [x] packages/archive/ Verzeichnis anlegen
        - catalog.toml erstellen (namespace: archive, type: archived)
        - README.md: "Diese Pakete sind noch nicht implementiert.
          Sie werden hierher verschoben bis ein echtes Repo existiert."

B2. [x] Folgende Pakete nach packages/archive/ verschieben:
        - apps/builder/     → in Container Manager aufgegangen, kein eigenes Paket mehr
        - apps/showcase/    → Desktop-interne Entwicklungs-Ansicht, kein Store-Paket
        - apps/profile/     → Desktop-intern (fs-desktop/crates/fs-profile)
        - apps/settings/    → Desktop-intern (fs-desktop/crates/fs-settings)

B3. [x] Folgende Pakete behalten, aber Katalog-TOMLs korrigieren:
        - apps/theme-app/   → nach archive/ (Theme = managers/theme)
        - apps/managers/    → aufgeteilt: managers/language, managers/theme, managers/icons,
                               managers/cursor, managers/container, managers/bots, managers/ai
        - apps/bots/        → Repo-Referenz fs-apps → fs-bots korrigiert

B4. [x] Package-Format upgraden: [[binaries]] einführen
        - Alle packages/apps/*/catalog.toml auf neues Format umgestellt (24 Dateien)
        - binary = "..." → [[binaries]] Array
        - bus = [...] → bus_messages = [...]
        - capabilities + bus_messages Felder überall ergänzt

B5. [x] Container-Pakete prüfen (packages/containers/)
        - Alle 10 bleiben: cryptpad, dragonfly, forgejo, openobserver, otel-collector,
          outline, postgres, pretix, umap, vikunja
        - Werden als erste installiert — später separat verfeinert

B6. [x] Store/ committen und pushen
```

---

## Phase C: Repositories anlegen und vorbereiten

> Ziel: Jedes Programm hat sein eigenes sauberes Repo auf GitHub, lokal geklont.

**Bereits vorhanden (kein Handlungsbedarf):**
```
fs-init, fs-node, fs-desktop, fs-store, fs-browser (✅ umbenannt),
fs-managers, fs-bots, fs-icons, fs-tasks, fs-inventory, fs-session,
fs-registry, fs-bus, fs-config, fs-libs, Store, fs-documentation
```

**Fehlende Repos — erstellen:**
```
C1. [ ] FreeSynergy/fs-db erstellen
        - DB-Abstraktion (SQLite via sea-orm, gemeinsam für alle Programme)
        - Workspace: eine Crate fs-db
        - Aus fs-libs extrahieren: src aus fs-libs/fs-db/

C2. [ ] FreeSynergy/fs-lenses erstellen
        - Lenses App (Informations-Betrachter)
        - Code-Basis: fs-apps/crates/fs-lenses/

C3. [ ] FreeSynergy/fs-ai erstellen
        - AI Runtime + Proxy (Schnittstelle zu fs-mistral und anderen LLMs)
        - Code-Basis: fs-apps/crates/fs-ai/ + fs-managers/ai/

C4. [ ] FreeSynergy/fs-container-app erstellen
        - Container-App Manager UI
        - Code-Basis: fs-apps/crates/fs-container-app/
```

**Lokales Klonen (fehlende Repos):**
```
C5. [ ] fs-bus lokal klonen nach /home/kal/Server/fs-bus/
C6. [ ] fs-config lokal klonen nach /home/kal/Server/fs-config/
C7. [ ] fs-tasks lokal klonen nach /home/kal/Server/fs-tasks/
        (Repo existiert auf GitHub, aber Inhalt prüfen)
```

**Alte Repos archivieren (auf GitHub als archived markieren):**
```
C8. [ ] FreeSynergy/Libs → archivieren (ersetzt durch fs-libs)
C9. [ ] FreeSynergy/Wiki.rs → archivieren (falls nicht mehr aktiv)
C10.[ ] FreeSynergy/Wiki.rs.Store → archivieren (falls nicht mehr aktiv)
```

---

## Phase D: fs-libs schrumpfen

> Ziel: fs-libs enthält nur echte, universelle Primitives.
> Alles Domain-spezifische geht ins zugehörige Programm-Repo.

**Bleibt in fs-libs (echte Primitives — alle Programme brauchen sie):**
```
fs-types    — FsValue, FsUrl, SemVer, LanguageCode, FsPort, FsTag
fs-error    — Basis-Fehler-Infrastruktur
fs-crypto   — Verschlüsselung (alle brauchen sie)
fs-health   — Health-Check-Trait (alle Services implementieren ihn)
fs-i18n     — i18n-Primitive (alle Programme sind mehrsprachig)
```

**Wandert in eigene Repos (bereits vorhanden):**
```
D1. [ ] fs-bus     → bereits eigenes Repo, aus fs-libs entfernen
D2. [ ] fs-config  → bereits eigenes Repo, aus fs-libs entfernen
D3. [ ] fs-db      → neues Repo (Phase C1), aus fs-libs extrahieren
```

**Wandert in Programm-Repos:**
```
D4. [ ] fs-auth, fs-federation  → nach fs-node
        (Node verwaltet Auth + Federation)

D5. [ ] fs-llm                  → nach fs-ai
        (AI Runtime braucht LLM-Abstraktion)

D6. [ ] fs-bot, fs-channel      → nach fs-bots
        (Bot-Runtime und Messenger-Adapter gehören zusammen)

D7. [ ] fs-ui, fs-components,
        fs-render, fs-theme     → nach fs-desktop
        (UI-Primitives gehören zum Desktop-Shell)

D8. [ ] fs-container            → nach fs-managers (container)
        (Container-Deployment-Logik gehört zum Container Manager)

D9. [ ] fs-pkg, fs-plugin-sdk,
        fs-plugin-runtime       → nach fs-store
        (Paket-Verwaltung gehört zur Store-Library)

D10.[ ] fs-sync                 → prüfen: wer nutzt es?
        Falls nur fs-store: nach fs-store
        Falls mehrere: in fs-libs lassen

D11.[ ] fs-sysinfo              → nach fs-node
        (System-Info ist Server-Aufgabe)

D12.[ ] fs-template             → prüfen: wer nutzt es?
        Falls nur ein Programm: dort hin

D13.[ ] fs-help                 → nach fs-desktop oder fs-store
        (Help-System gehört zur UI-Schicht)

D14.[ ] fs-bridge-sdk           → löschen (Adapter-Pattern ersetzt Bridges)
        Erst löschen wenn fs-registry voll funktioniert

D15.[ ] fs-core                 → prüfen: was ist drin?
        Falls nur Glue-Code: auflösen

D16.[ ] fs-libs committen + pushen (nach allen Migrationen)
```

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

```
E01.[ ] fs-config   — Konfig-Parsing (klein, klar definiert)
E02.[ ] fs-bus      — Message Bus (Platform-Service, alle brauchen ihn)
E03.[ ] fs-db       — DB-Abstraktion (nach Phase C1 + D3)
E04.[ ] fs-inventory — Installiertes verwalten (bereits sauber, Integration prüfen)
E05.[ ] fs-session  — Session-Management (bereits sauber, Integration prüfen)
E06.[ ] fs-registry — Capability-Registry (bereits sauber, Integration prüfen)
E07.[ ] fs-init     — Bootstrap (klein, kritisch)
E08.[ ] fs-icons    — Icon-Verwaltung
E09.[ ] fs-browser  — VORLAGE für alle anderen Programme
        → erster vollständiger Workflow: Design → OOP → Tests → Clippy → Push
E10.[ ] fs-store    — Store-Library + Store-App + Store-CLI
        → besonders wichtig: StoreReader liest Store/-Katalog korrekt (Test!)
E11.[ ] fs-bots     — Bot-Runtime (Workspace mit mehreren Binaries)
        → Struktur: broadcast-bot, menu-bot, calendar-bot, ... je nach Messenger
E12.[ ] fs-managers — Alle 7 Manager als Workspace
        → language, theme, icons, cursor, container, bots, ai
        → Reihenfolge: language → theme → icons → cursor → container → bots → ai
E13.[ ] fs-lenses   — Lenses App (nach Phase C2)
E14.[ ] fs-tasks    — Tasks App
E15.[ ] fs-container-app — Container-App Manager UI (nach Phase C4)
E16.[ ] fs-ai       — AI Runtime + Proxy (nach Phase C3)
E17.[ ] fs-node     — Server (Auth, S3, Federation, externer Zugriff)
        → größtes und komplexestes Repo, erst wenn alle Libs stabil sind
E18.[ ] fs-desktop  — Desktop-Shell
        → nach fs-node (braucht Node für Authentifizierung)
```

---

## Phase F: Integration

> Ziel: Die Programme reden miteinander.
> Erst wenn alle Repos aus Phase E ihren Repo-Gate bestanden haben.

```
F1. [ ] fs-store ↔ Store/ Kompatibilitäts-Test
        - StoreReader liest alle TOML-Kataloge aus Store/ korrekt
        - Paket-Parsing: alle Felder werden erkannt (auch [[binaries]])
        - Integrations-Test: store-cli list gibt tatsächlich Pakete aus

F2. [ ] Manager → fs-inventory
        - Jeder Manager schreibt nach erfolgreicher Aktion ins Inventory
        - Test: Theme-Manager installiert Theme → fs-inventory zeigt es als installiert

F3. [ ] fs-bus verdrahten
        - fs-inventory subscribt auf installer::* Bus-Messages
        - fs-registry subscribt auf service::* Bus-Messages
        - Test: programm startet → registriert sich im Bus → Registry kennt es

F4. [ ] fs-session in Desktop einbauen
        - Desktop öffnet Programm → Session-Eintrag wird angelegt
        - Minimize → ProgramState::Minimized
        - Restore → bestehendes Fenster wird aufgemacht (kein Neustart)

F5. [ ] fs-registry → Adapter-Lookup
        - Service registriert Capability beim Start
        - Bus fragt Registry: "Wer kann Rolle iam?"
        - Antwort: Kanidm auf http://kanidm:8443

F6. [ ] End-to-End Test: Paket installieren
        - fs-store-app: Paket auswählen
        - Download-URL aus Store/-Katalog
        - Binary wird heruntergeladen + Hash-Verifikation
        - Eintrag in fs-inventory
        - Service startet → registriert sich in fs-registry
        - Bus-Subscriber können ihn finden
```

---

## Nächste Gespräche

> Diese Themen müssen besprochen werden bevor sie umgesetzt werden können.
> Kein Code ohne Gespräch und Entscheidung.

```
G1. fs-node Architektur (deep-dive)
    - Was genau macht der Node? Welche Module hat er?
    - Auth (Kanidm-Integration), S3, Federation, externer Desktop-Zugriff
    - Wie trennen wir die Verantwortlichkeiten sauber?

G2. Desktop Architektur
    - App-Lifecycle: wie wird ein Programm gestartet/minimiert/beendet?
    - Fenster-Management: wer verwaltet was?
    - Verbindung zu fs-session

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
A  Dokumentation aktualisieren          ← jetzt
B  Store bereinigen                     ← jetzt
C  Repositories anlegen                 ← jetzt
D  fs-libs schrumpfen                   ← parallel zu C
E  Programme einzeln sauber machen      ← E01–E09 zuerst
F  Integration                          ← nach E
G  Gespräche (parallel zu E+F führen)   ← laufend
—  Archiv-Phasen                        ← wenn Zeit da ist
```
