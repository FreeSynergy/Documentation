# FreeSynergy — Build Plan

[← Zurück zum Index](../INDEX.md)

---

## TODO-Regeln (für Claude-Sessions)

- **TODO.md immer vollständig lesen** zu Beginn jeder Session — auch wenn es Token kostet.
  Nur so gibt es einen vollständigen Überblick ohne Missverständnisse.
- **Erledigte Items löschen** — kein [x] behalten. Löschen spart Token in zukünftigen Sessions.
  Abgeschlossene Phasen werden auf eine Zeile komprimiert.
- **Vor jeder Aktion:** Was braucht dieses Modul? Store als Single Source of Truth prüfen.

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

## Phase C: Repositories anlegen und vorbereiten

> Ziel: Jedes Programm hat sein eigenes sauberes Repo auf GitHub, lokal geklont.

**Lokal vorhanden + GitHub ✅ (kein Handlungsbedarf):**
```
fs-init, fs-node, fs-desktop, fs-store, fs-browser,
fs-managers, fs-bots, fs-icons, fs-inventory, fs-session,
fs-registry, fs-bus, fs-config, fs-libs, Store, fs-documentation
```

**Neu angelegt ✅:** fs-db, fs-lenses, fs-ai, fs-container-app, fs-tasks, fs-auth, fs-i18n, fs-info, fs-theme, fs-ui, fs-components
**Archiviert ✅:** Libs, Wiki.rs, Wiki.rs.Store

**Neue Repos — noch anzulegen:**
```
C20.[ ] FreeSynergy/fs-federation erstellen — Föderations-Logik
        - Node-zu-Node-Kommunikation, Domain-Auth, Invite-System
        - Groß genug für eigenes Repo mit eigener Versionierung

C21.[ ] FreeSynergy/fs-llm erstellen — LLM-Abstraktions-Layer
        - Schnittstelle zu Mistral.rs und anderen LLM-Backends
        - Wird von fs-ai UND fs-bots gebraucht

C22.[ ] FreeSynergy/fs-channel erstellen — Messaging-Kanal-Abstraktionen
        - Kanal-Adapter: Matrix, Telegram, Signal, ...
        - Basis für fs-bots und zukünftige Messaging-Apps

C23.[ ] FreeSynergy/fs-packages erstellen — Paket- und Plugin-Verwaltung
        - Kombiniert: fs-pkg (Paket-Format), fs-plugin-sdk, fs-plugin-runtime
        - Wird von fs-store UND fs-init gebraucht (daher nicht in fs-store)

C24.[ ] FreeSynergy/fs-container erstellen — Container-Deployment-Library
        - Container starten, stoppen, verwalten (Library, nicht UI)
        - Wird von fs-managers/container UND fs-node gebraucht
        - Achtung: NICHT fs-container-app (das ist die UI, bereits C4)
```

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
```

---

**Bereits erledigt ✅:** fs-bus | fs-config | fs-db | fs-auth | fs-federation | fs-i18n | fs-theme | fs-ui | fs-components | fs-render | fs-llm | fs-channel → jeweils eigenes Repo

---

**Eigene Repos — Netzwerk / Protokoll-Schicht:**
```
D13.[ ] fs-bot → nach fs-bots
        Was: Bot-Runtime-Logik (Nachrichten empfangen, verarbeiten, antworten)
        Wer braucht es: NUR fs-bots
        Begründung: echte 1:1-Zuordnung — Bot-Runtime gehört zum Bot-Programm
```

---

**Eigene Repos — Infrastruktur-Dienste (laufen als Daemons, antworten auf Bus):**
```
D14.[ ] fs-sysinfo → eigenes Repo fs-info (Phase C13)
        Was: System-Info (Speicher, CPU, Disk, laufende Services)
        Wer braucht es: Store (ist Platz da?), Widgets, Desktop, Manager
        Warum eigen: mehrere Nutzer — NICHT nach fs-node
        Ob Daemon, Bus-Subscriber oder Library: offen → siehe G8
```

---

**Paket- und Plugin-Verwaltung:**
```
D15.[ ] fs-pkg, fs-plugin-sdk, fs-plugin-runtime → eigenes Repo fs-packages (Phase C23)
        Was: Paket-Format, Plugin-Schnittstelle, Plugin-Laufzeitumgebung
        Wer braucht es: fs-store primär, aber auch fs-init (installiert Pakete beim Bootstrap)
        Warum eigen: die drei gehören fachlich zusammen (Paket-Verwaltungs-Schicht)
        Warum nicht in fs-store: fs-init braucht sie auch, vor dem Store

D16.[ ] fs-container → eigenes Repo (Phase C24)
        Was: Container-Deployment-Logik (starten, stoppen, verwalten)
        Wer braucht es: fs-managers/container UND fs-node (Bootstrap)
        Warum eigen: mehrere Nutzer
        (Achtung: fs-container = Library. fs-container-app = UI. Zwei verschiedene Repos.)
```

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

**Noch zu prüfen (Code lesen, dann entscheiden):**
```
D20.[ ] fs-sync → Code lesen: was synchronisiert es und für wen?
        Falls nur fs-store: nach fs-store
        Falls mehrere oder unklar: eigenes Repo

D21.[ ] fs-template → Code lesen: wer nutzt es?
        Falls nur ein Programm: dorthin
        Falls mehrere: eigenes Repo oder in fs-libs lassen

D22.[ ] fs-help → Code lesen: ist es eine Library oder ein Programm?
        Falls Library für alle: eigenes Repo (Phase C25)
        Falls Desktop-spezifisch: nach fs-desktop

D23.[ ] fs-core → Code lesen: was ist drin?
        Vermutung: nur Glue-Code → auflösen und verteilen
```

---

**Löschen:**
```
D24.[ ] fs-bridge-sdk → löschen
        Adapter-Pattern hat Bridges ersetzt — kein Nutzen mehr
        Erst löschen wenn fs-registry läuft und alle Bridges migriert sind
```

---

**Abschluss:**
```
D25.[ ] fs-libs committen + pushen
        Erst wenn alle Migrationen abgeschlossen und alle abhängigen Repos angepasst sind
        Erwartetes Ergebnis: fs-libs hat nur noch fs-types, fs-error, fs-crypto, fs-health
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

**Erledigt ✅:** fs-config, fs-bus, fs-db, fs-inventory, fs-session, fs-registry, fs-init, fs-icons, fs-browser, fs-store

```
E11.[ ] fs-bots     — Bot-Runtime (Workspace mit mehreren Binaries)
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
G1. fs-auth Design-Entscheidung + fs-node Architektur
    AUTH — Entscheidung nötig bevor Code geschrieben wird:

    Option A: fs-auth = Adapter-Schicht
      - fs-auth definiert Traits + eigene Login-UI + CLI
      - Kanidm ist eine Implementierung davon (austauschbar)
      - Pro: flexibel, Kanidm kann später ersetzt werden
      - Con: wir reimplementieren Auth-Konzepte nach; mehr Code für weniger Mehrwert
        wenn wir Kanidm sowieso immer nutzen

    Option B: Kanidm direkt integrieren (kein Adapter)
      - Kanidm wird direkt als Auth-Backend eingebunden
      - Pro: nutzt Kanidm voll aus (PAM, SCIM intern, OIDC/LDAP)
        PAM = OS-Level-Auth (seltenes Feature, sehr mächtig für ein Platform-OS)
      - Con: tightly coupled; Wechsel zu anderem Backend wäre großer Aufwand
      - Frage: Wollen wir Kanidm langfristig? Oder ist das nur der erste Schritt?

    Zu bedenken:
      - Kanidm kann PAM (OS-Ebene) + SCIM (intern, extern in Entwicklung)
      - Kanidm wird aktiv entwickelt, aber unsicher wie groß das Projekt bleibt
      - FreeSynergy ist ein Platform-OS — Auth ist keine optionale Komponente

    NODE — nach Auth-Entscheidung:
      - Was macht der Node genau? (S3, externer Desktop-Zugriff, Orchestrierung)
      - Wie viel bleibt im Node, was geht in eigene Module?
      - Verbindung zu fs-auth, fs-federation, fs-registry

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
C  Repositories anlegen                 ← in Arbeit (C11–C24 offen)
D  fs-libs schrumpfen                   ← parallel zu C
E  Programme einzeln sauber machen      ← E03–E18 offen
F  Integration                          ← nach E
G  Gespräche (parallel zu E+F führen)   ← laufend
—  Archiv-Phasen                        ← wenn Zeit da ist
```
