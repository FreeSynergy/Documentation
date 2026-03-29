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

```
[ ] D24: fs-bridge-sdk löschen
        Erst wenn fs-registry läuft und alle Nutzer auf Adapter migriert

[ ] H2:  fs-core Standalone-Repo bereinigen
        /home/kal/Server/fs-core/ — Überbleibsel, nicht mehr genutzt
        Aktion: GitHub archivieren + lokales Verzeichnis löschen
```

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

## fs-bootc (NEU — program)

> Fedora bootc OS-Images: fs-server + fs-workstation

```
OOP & Design
[ ] ImageVariant-Trait: build() / push() / name()
[ ] ServerImage + WorkstationImage: implementieren ImageVariant
[ ] Immer gegen Interface

Repo
[ ] GitHub Repo anlegen: git@github.com:FreeSynergy/fs-bootc.git
[ ] Lokal anlegen: /home/kal/Server/fs-bootc/
[ ] CLAUDE.md / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile: fs-server (FROM fedora-bootc:41, kein Wayland)
[ ] Containerfile: fs-workstation (FROM fedora-bootc:41, Wayland-Pakete)
[ ] Butane/Ignition-Config für fs-init-Integration
[ ] GitHub Actions: build + push zu ghcr.io/freesynergy/ bei Tag

CLI
[ ] fs-bootc build server|workstation
[ ] fs-bootc push

Dokumentation
[ ] Doku-Seite: Image-Aufbau, Varianten, bootc install to-disk
[ ] commit + push
```

---

## fs-db-engine-sqlite (NEU — adapter)

> SqliteEngine: implementiert DbEngine-Trait aus fs-db

```
OOP & Design
[✓] Adapter Pattern: SqliteEngine wraps sqlx + SQLite
[✓] SqliteEngine implementiert DbEngine-Trait vollständig
[ ] Registriert Capability "db.engine.sqlite" in fs-registry
[✓] Immer gegen Interface — Consumer kennt nur DbEngine-Trait

Repo
[✓] GitHub Repo anlegen: git@github.com:FreeSynergy/fs-db-engine-sqlite.git
[✓] Lokal anlegen: /home/kal/Server/fs-db-engine-sqlite/
[✓] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[✓] Containerfile

Code-Qualität
[✓] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für alle user-facing Fehlermeldungen (Verbindungsfehler, Migration, etc.)
[✓] cargo clippy: 0 Fehler
[✓] cargo fmt --check: sauber
[✓] cargo test: connect / migrate / query / health getestet
[ ] cargo build --release: fehlerfrei

API
[✓] gRPC (tonic): open / close / migrate / execute / health
[✓] Proto-Datei: db_engine.proto (geteilt mit fs-db-engine-postgres)
[✓] REST (axum): GET /health, POST /migrate, POST /execute
[✓] OpenAPI (utoipa): auto-generiert

Dokumentation
[ ] Doku-Seite: SqliteEngine-Impl, Capability-Registration, API
[ ] commit + push
```

---

## fs-db-engine-postgres (NEU — adapter)

> PostgresEngine: implementiert DbEngine-Trait aus fs-db

```
OOP & Design
[ ] Adapter Pattern: PostgresEngine wraps SeaORM + PostgreSQL
[ ] PostgresEngine implementiert DbEngine-Trait vollständig
[ ] Registriert Capability "db.engine.postgres" in fs-registry
[ ] Immer gegen Interface

Repo
[ ] GitHub Repo anlegen: git@github.com:FreeSynergy/fs-db-engine-postgres.git
[ ] Lokal anlegen: /home/kal/Server/fs-db-engine-postgres/
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für alle user-facing Fehlermeldungen
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: connect / migrate / query / health getestet
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC (tonic): connect / disconnect / migrate / query / health
[ ] REST (axum): GET /health, POST /migrate, POST /query
[ ] OpenAPI (utoipa): auto-generiert

Dokumentation
[ ] Doku-Seite: PostgresEngine-Impl, Capability-Registration, API
[ ] commit + push
```

---

## fs-llm-mistral (NEU — adapter)

> MistralAdapter: implementiert LlmAdapter-Trait aus fs-llm

```
OOP & Design
[ ] Adapter Pattern: MistralAdapter wraps fs-mistral (Fork)
[ ] MistralAdapter implementiert LlmAdapter-Trait
[ ] Registriert Capability "llm.mistral" in fs-registry
[ ] Immer gegen Interface

Repo
[ ] GitHub Repo anlegen: git@github.com:FreeSynergy/fs-llm-mistral.git
[ ] Lokal anlegen: /home/kal/Server/fs-llm-mistral/
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für Fehlermeldungen + Status-Texte
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: LlmAdapter-Trait getestet (Mock)
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC (tonic): complete / embed / stream / list-models
[ ] REST (axum): POST /complete, POST /embed, POST /stream, GET /models
[ ] OpenAPI (utoipa): auto-generiert

Dokumentation
[ ] Doku-Seite: MistralAdapter-Impl, Capability, API
[ ] commit + push
```

---

## fs-llm-openai (NEU — adapter)

> OpenAiAdapter: implementiert LlmAdapter-Trait aus fs-llm

```
OOP & Design
[ ] Adapter Pattern: OpenAiAdapter wraps OpenAI API
[ ] OpenAiAdapter implementiert LlmAdapter-Trait
[ ] Registriert Capability "llm.openai" in fs-registry
[ ] Immer gegen Interface

Repo
[ ] GitHub Repo anlegen: git@github.com:FreeSynergy/fs-llm-openai.git
[ ] Lokal anlegen: /home/kal/Server/fs-llm-openai/
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für Fehlermeldungen + Status-Texte
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: LlmAdapter-Trait getestet (Mock)
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC (tonic): complete / embed / stream / list-models
[ ] REST (axum): POST /complete, POST /embed, POST /stream, GET /models
[ ] OpenAPI (utoipa): auto-generiert

Dokumentation
[ ] Doku-Seite: OpenAiAdapter-Impl, Capability, API
[ ] commit + push
```

---

## fs-channel-matrix (NEU — adapter)

> MatrixAdapter: implementiert ChannelAdapter-Trait aus fs-channel

```
OOP & Design
[ ] Adapter Pattern: MatrixAdapter wraps matrix-sdk (fs-tuwunel Fork)
[ ] MatrixAdapter implementiert ChannelAdapter-Trait
[ ] Registriert Capability "channel.matrix" in fs-registry
[ ] Immer gegen Interface

Repo
[ ] GitHub Repo anlegen: git@github.com:FreeSynergy/fs-channel-matrix.git
[ ] Lokal anlegen: /home/kal/Server/fs-channel-matrix/
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für alle user-facing Texte + Fehlermeldungen
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: ChannelAdapter-Trait getestet
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC (tonic): send / receive / subscribe / list-rooms / join / leave
[ ] REST (axum): POST /messages, GET /rooms, POST /rooms/{id}/join
[ ] OpenAPI (utoipa): auto-generiert

Spezifisch
[ ] N0: PostgreSQL State Store (Blocker: fs-db-engine-postgres fertig)
        Bis dann: In-Memory-Store
        Bekannter Bug: matrix feature → recursion overflow in rustc ≥1.94 (upstream)

Dokumentation
[ ] Doku-Seite: MatrixAdapter-Impl, State-Store, Capability, API
[ ] commit + push
```

---

## fs-channel-telegram (NEU — adapter)

> TelegramAdapter: implementiert ChannelAdapter-Trait aus fs-channel

```
OOP & Design
[ ] Adapter Pattern: TelegramAdapter wraps Telegram Bot API
[ ] TelegramAdapter implementiert ChannelAdapter-Trait
[ ] Registriert Capability "channel.telegram" in fs-registry
[ ] Immer gegen Interface

Repo
[ ] GitHub Repo anlegen: git@github.com:FreeSynergy/fs-channel-telegram.git
[ ] Lokal anlegen: /home/kal/Server/fs-channel-telegram/
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für alle user-facing Texte + Fehlermeldungen
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: ChannelAdapter-Trait getestet
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC (tonic): send / receive / subscribe / list-rooms / join / leave
[ ] REST (axum): POST /messages, GET /chats
[ ] OpenAPI (utoipa): auto-generiert

Dokumentation
[ ] Doku-Seite: TelegramAdapter-Impl, Capability, API
[ ] commit + push
```

---

---

# Gruppe B — fs-libs (Monorepo, 4 Crates, kein Containerfile)

> Compile-Time Dependencies. Kein Container, kein Daemon, keine API.
> fs-libs hat 4 universelle Primitives: fs-types, fs-error, fs-crypto, fs-health

---

## fs-types (Crate in fs-libs)

> Universelle Basis-Typen: FsValue, FsUrl, SemVer, LanguageCode, FsPort, FsTag

```
OOP & Design
[ ] Value Object Pattern: alle Typen immutable, Verhalten im Typ
[ ] Newtype-Wrapper: kein nackter String / u32 durchreichen
[ ] Traits: Display, From, TryFrom, Serialize, Deserialize für alle Typen
[ ] Immer gegen Interface — Consumer nutzt Trait-Bounds

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für alle user-facing Fehlermeldungen (TryFrom-Fehler, Display-Texte)
    Auch CLI- und Library-Nutzer sehen diese Texte → FTL Pflicht
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: alle Typen round-trip + Display + TryFrom getestet
[ ] cargo build --release: fehlerfrei

Spezifisch
[ ] u16 für Versionsnummern (nicht u32)
[ ] FsUrl: valide URL, kein nackter String
[ ] SemVer: major.minor.patch, Vergleich implementiert
[ ] LanguageCode: BCP-47
[ ] FsPort: 1–65535

Dokumentation
[ ] Doku-Seite: alle Typen, Traits, Verwendungsbeispiele
[ ] commit + push
```

---

## fs-error (Crate in fs-libs)

> Basis-Fehler-Infrastruktur

```
OOP & Design
[ ] Composite Error Pattern: FsError wraps domain errors
[ ] FsError-Trait: code / message / cause / severity
[ ] Jede Fehler-Domäne implementiert FsError

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für alle user-facing Fehlermeldungen — auch CLI + Library-Nutzer sehen sie
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

Spezifisch
[ ] FsError: code (String), message (FTL-Key), severity (Info/Warn/Error/Fatal)
[ ] From-Impl für häufige std-Fehler
[ ] thiserror als einzige externe Dependency

Dokumentation
[ ] Doku-Seite: FsError-Trait, Fehler-Hierarchie, Verwendung
[ ] commit + push
```

---

## fs-crypto (Crate in fs-libs)

> Verschlüsselung für alle FS-Dienste

```
OOP & Design
[ ] Strategy Pattern: CryptoProvider-Trait, Algorithmus austauschbar
[ ] CryptoProvider-Trait: encrypt / decrypt / sign / verify / hash
[ ] Immer gegen Interface

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für Fehlermeldungen (auch CLI-Nutzer sehen diese)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: alle Crypto-Ops round-trip getestet
[ ] cargo build --release: fehlerfrei

Spezifisch
[ ] HMAC-Impl (Invite-Token in fs-node)
[ ] age-Verschlüsselung (Invite-Bundle in fs-node)
[ ] Kein unsafe code

Dokumentation
[ ] Doku-Seite: CryptoProvider-Trait, Algorithmen, Sicherheitshinweise
[ ] commit + push
```

---

## fs-health (Crate in fs-libs)

> Health-Check-Trait für alle Services

```
OOP & Design
[ ] Observer Pattern: HealthMonitor beobachtet Services
[ ] HealthCheck-Trait: check() → HealthStatus
[ ] HealthStatus: Healthy / Degraded / Unhealthy + Details

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für Status-Texte (auch CLI-Ausgaben)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

Dokumentation
[ ] Doku-Seite: HealthCheck-Trait, HealthStatus, Verwendung
[ ] commit + push
```

---

---

# Gruppe C — Infrastruktur-Libraries

---

## fs-bus (program)

> Async Event Bus: Topic-Routing, Retry, Tera-Transforms

```
OOP & Design
[ ] Pub-Sub + Chain of Responsibility (Transforms als Chain)
[ ] BusPublisher-Trait: publish(topic, payload) → Result
[ ] BusSubscriber-Trait: subscribe(pattern) → Stream
[ ] MessageTransform-Trait: transform(msg) → msg
[ ] RetryPolicy-Trait: Strategy Pattern (exponential / fixed / none)
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle Fehlermeldungen + Status-Texte (CLI + UI + Logs)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: publish / subscribe / retry / transform getestet
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC (tonic): publish / subscribe / ack / list-topics / unsubscribe
[ ] Proto-Datei: bus.proto
[ ] REST (axum): POST /publish, GET /topics, POST /subscribe
[ ] OpenAPI (utoipa): auto-generiert

Spezifisch
[ ] G3: Bus-Namespaces dokumentieren (nach Architektur-Gespräch)
[ ] Topic-Format: {domain}.{action}.{qualifier}
[ ] Rechte: wer darf welche Topics publizieren / subscriben

CLI
[ ] fs-bus publish {topic} {payload}
[ ] fs-bus subscribe {pattern}
[ ] fs-bus list-topics
[ ] fs-bus status

Dokumentation
[ ] Doku-Seite: Traits, Topic-Format, Namespaces, API, CLI
[ ] commit + push
```

---

## fs-config (library, kein Container)

> TOML Config Loader/Saver — reine Library, kein Daemon

```
OOP & Design
[ ] Repository Pattern: ConfigRepository lädt/speichert
[ ] ConfigLoader-Trait: load() → Config, save(config) → Result
[ ] ConfigValidator-Trait: validate(config) → ValidationResult
[ ] ConfigAutoRepair-Trait: repair(config) → Config
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Kein Containerfile (reine Library)

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Validierungsfehler + Fehlermeldungen (auch CLI-Nutzer sehen sie)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: load / save / validate / repair getestet
[ ] cargo build --release: fehlerfrei

Spezifisch
[ ] Config-Changes via Bus: config::changed::* Topics publizieren
[ ] Kein Daemon — Programme subscriben auf config::changed::*

Dokumentation
[ ] Doku-Seite: ConfigLoader-Trait, Validierung, Auto-Repair, Bus-Integration
[ ] commit + push
```

---

## fs-db (library, kein Container)

> DbEngine-Trait — reine Abstraktion, kein eigener Container

```
OOP & Design
[ ] Strategy Pattern: DbEngine ist austauschbar
[ ] DbEngine-Trait: connect / disconnect / migrate / query / health
[ ] MigrationRunner-Trait: run() / rollback() / status()
[ ] Immer gegen Interface — Consumer kennt nur DbEngine-Trait
[ ] Konkrete Engines kommen als eigene Repos (fs-db-engine-sqlite, fs-db-engine-postgres)

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Kein Containerfile (reine Library)

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle DB-Fehlermeldungen (auch CLI-Nutzer sehen sie)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: Trait-Definitionen + Stub-Impl getestet
[ ] cargo build --release: fehlerfrei

Spezifisch
[ ] G7: DbEngine-Trait API final festlegen (vor Umsetzung Gespräch nötig)
[ ] Feature-Flag: sqlite (default, lädt fs-db-engine-sqlite), postgres
[ ] Kein direktes SeaORM / sqlx in Consumer-Crates
[ ] Migration-Dateien: je Engine in ihrem eigenen Repo

Dokumentation
[ ] Doku-Seite: DbEngine-Trait, MigrationRunner, Wie Engines geladen werden, API
[ ] commit + push
```

---

## fs-i18n (program)

> i18n-Service: FTL laden, übersetzen, Hotplugging, OCI Artifact Pull

```
OOP & Design
[ ] Observer Pattern: inotify beobachtet Locales-Pfad
[ ] I18nLoader-Trait: load(lang) / available() / t(key, args) → String
[ ] ArtifactPuller-Trait: pull(lang, source) → Result
[ ] HotplugObserver: on_file_changed(path) → reload
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile (fs-i18n ist ein Service)

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: load / t() / hotplug / fallback / global-vs-per-package getestet
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC (tonic): translate / available-languages / reload / pull-artifact
[ ] Proto-Datei: i18n.proto
[ ] REST (axum): GET /translate, GET /languages, POST /reload
[ ] OpenAPI (utoipa): auto-generiert

Spezifisch
[ ] inotify: FTL hotplugging ohne Restart
[ ] OCI Artifact Pull + Extraktion nach locales/
[ ] Global vs. Per-Paket: translate(key, lang, package_id: Option)
[ ] Weblate-kompatible Struktur: locales/{lang}/{programm}.ftl
[ ] common.ftl: error-*, action-*, label-*, status-*, phrase-* (80+ Keys)
[ ] FTL-Dateien für neue Repos anlegen: fs-bus, fs-db, fs-registry, fs-session,
    fs-inventory, fs-info, fs-help, fs-sync, fs-bootc — je en + de

CLI
[ ] fs-i18n translate {key} {lang}
[ ] fs-i18n available
[ ] fs-i18n pull {lang}
[ ] fs-i18n reload

Dokumentation
[ ] Doku-Seite: I18nLoader-Trait, Hotplugging, OCI Artifacts, Global vs. Per-Paket, API, CLI
[ ] commit + push
```

---

## fs-theme (program)

> Theme-Definitionen + Loader

```
OOP & Design
[ ] Repository Pattern (ThemeRepository) + Factory (ThemeFactory)
[ ] ThemeLoader-Trait: load(id) / list() / active() / apply(theme)
[ ] FsTheme: Farben, Fonts, Abstände, Icon-Set, Cursor-Set — alles typisiert
[ ] ThemeValidator-Trait: validate(theme) → Result
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Theme-Namen + Beschreibungen + Fehlermeldungen (CLI + UI)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: load / validate / apply getestet
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC (tonic): load / list / active / apply / validate
[ ] REST (axum): GET /themes, GET /themes/{id}, POST /themes/{id}/apply
[ ] OpenAPI (utoipa): auto-generiert

Spezifisch
[ ] Palette-Typen vollständig (src/palette.rs)
[ ] Bus-Integration: theme::changed Event publizieren
[ ] Theme als Artifact nachladen (global oder per-Paket)

CLI
[ ] fs-theme list / active / apply {id}

Dokumentation
[ ] Doku-Seite: ThemeLoader-Trait, FsTheme-Format, Artifacts, API, CLI
[ ] commit + push
```

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
[ ] cargo build --release: fehlerfrei

Spezifisch
[x] AppContext: Locale + Theme + FeatureFlags — durch RenderEngine::set_context durchgereicht
[x] 3D-Erweiterungs-API: Fs3dExtension + Fs3dDescriptor (optional, FsView-Basis unverändert)

Dokumentation
[ ] Doku-Seite: alle Traits, FsView-Pattern (Warum hier?), AppContext, 3D-Erweiterung
[ ] commit + push
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

## fs-gui-engine-bevy (adapter)

> Bevy-Implementierung von fs-render (3D-fähig)

```
OOP & Design
[ ] Adapter Pattern: BevyEngine adaptiert Bevy ECS für fs-render
[ ] BevyEngine implementiert RenderEngine-Trait
[ ] WorkspaceScene: Resource (Bevy ECS), Builder Pattern
[ ] AnimatedBackground: Component (Particles / Gradient / WaveField / Static)
[ ] 3D-Erweiterungs-API: zusätzlich, stört FsView-Basis nicht
[ ] Registriert Capability "render.engine.bevy" in fs-registry
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für Fehlermeldungen
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

Spezifisch
[ ] WorkspaceLayout: Grid / Circular / Freeform
[ ] FsRenderPlugin: Bevy Plugin, korrekt registriert

Dokumentation
[ ] Doku-Seite: BevyEngine-Impl, WorkspaceScene, AnimatedBackground, 3D-API, Capability
[ ] commit + push
```

---

## fs-web-engine (library, kein Container)

> Browser-Engine-Abstraktions-Traits

```
OOP & Design
[ ] Abstract Factory: WebEngine erstellt WebViews
[ ] WebEngine-Trait: create_view / capabilities / version
[ ] WebView-Trait: navigate / reload / go_back / execute_js
[ ] WebPlugin-Trait: init / handle_request / name
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Kein Containerfile (reine Library)

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] Keine FTL (reine Trait-Library)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: StubWebEngine + Navigation getestet
[ ] cargo build --release: fehlerfrei

Dokumentation
[ ] Doku-Seite: WebEngine-Trait, WebView, WebPlugin, Capabilities
[ ] commit + push
```

---

## fs-web-engine-servo (adapter)

> Servo-Implementierung von fs-web-engine (SpiderMonkey JS, MPL-2.0)

```
OOP & Design
[ ] Adapter Pattern: ServoWebEngine adaptiert Servo für fs-web-engine
[ ] ServoWebEngine implementiert WebEngine-Trait
[ ] ServoPluginRegistry: Composite Pattern für WASM-Plugins
[ ] Request-Interception: Decorator Pattern
[ ] Registriert Capability "web.engine.servo" in fs-registry
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md (MPL-2.0!) / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für Fehlermeldungen
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

Spezifisch
[ ] Feature-Flag: servo (optional) — ohne Flag: StubWebEngine
[ ] WASM-Plugin-API via fs-plugin-sdk

Dokumentation
[ ] Doku-Seite: ServoWebEngine-Impl, Lizenz (MPL-2.0!), Plugin-API, Capability
[ ] commit + push
```

---

## fs-ui (library)

> UI-Basis-Komponenten — Migration von Dioxus zu fs-render

```
OOP & Design
[ ] Component Pattern: jede Komponente ist ein Objekt
[ ] Alle Komponenten implementieren FsView-Trait aus fs-render
[ ] Kein direktes iced / Dioxus in Komponenten
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] H11: Cargo.toml description → nicht mehr "Dioxus UI component library"
[ ] Kein Containerfile (Library)

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für alle user-facing Komponentenbeschriftungen
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

Spezifisch
[ ] Alle Dioxus-Komponenten auf fs-render FsView-Trait migrieren
[ ] Dioxus-Dependency vollständig entfernen

Dokumentation
[ ] Doku-Seite: Komponenten-Übersicht, FsView-Pattern, Migration-Status
[ ] commit + push
```

---

## fs-components (library)

> Wiederverwendbare App-Komponenten

```
OOP & Design
[ ] Component + Composite Pattern (Komponenten-Baum)
[ ] Alle Komponenten implementieren FsView-Trait
[ ] nav.rs: NavItem / NavGroup / NavBar als eigene Objekte — kein Monolith
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Kein Containerfile (Library)

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für alle user-facing Texte
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

Spezifisch
[ ] H5: src/nav.rs aufteilen (1204 Zeilen)
        → nav/item.rs + nav/group.rs + nav/bar.rs + nav/breadcrumb.rs
[ ] Dioxus-Dependency vollständig entfernen
[ ] Alle Komponenten auf FsView-Trait migrieren

Dokumentation
[ ] Doku-Seite: Komponenten-Katalog, FsView-Pattern
[ ] commit + push
```

---

---

# Gruppe E — Services

---

## fs-auth (program)

> 4 Protokoll-Traits + Kanidm-Implementierung

```
OOP & Design
[ ] Strategy Pattern: Auth-Backends austauschbar
[ ] OAuthProvider-Trait: authorize / token / refresh / revoke
[ ] ScimProvider-Trait: create_user / update_user / delete_user / list_users
[ ] SsoProvider-Trait: sso_login / validate_session / logout
[ ] PamProvider-Trait: pam_authenticate / pam_authorize
[ ] KanidmBackend: Adapter Pattern, implementiert alle 4 Traits
[ ] Immer gegen Interface — nie Kanidm direkt aufrufen

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Login / Logout / Fehler / Status (UI + CLI + API-Responses)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: alle 4 Protokoll-Traits + KanidmBackend getestet
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC: login / logout / provision / validate / capabilities
[ ] Proto-Datei: auth.proto
[ ] REST: POST /login, POST /logout, GET /capabilities, POST /provision
[ ] OpenAPI: auto-generiert

UI
[ ] LoginView: FsView-Trait, view.rs (Bindeglied) — nur diese Datei importiert fs-render
[ ] iced via fs-gui-engine-iced

CLI
[ ] fs-auth login / logout / status / provision

Bus
[ ] auth.login / auth.logout / auth.provision Events publizieren

Dokumentation
[ ] Doku-Seite: 4 Protokoll-Traits, KanidmBackend, LoginView-Pattern, API, CLI, Bus
[ ] commit + push
```

---

## fs-registry (program)

> Welche Services + Capabilities laufen?

```
OOP & Design
[ ] Registry Pattern
[ ] ServiceRegistry-Trait: register / deregister / lookup / list / health
[ ] CapabilityIndex: welcher Service bietet welche Capability
[ ] endpoint_for_capability(cap) → Option<String>
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Fehlermeldungen + Status-Texte (CLI + API-Responses)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: register / deregister / lookup / capability-lookup getestet
[ ] cargo build --release: fehlerfrei

API
[ ] G8: Daemon vs. Bus-Subscriber (vor Umsetzung klären)
[ ] gRPC: register / deregister / lookup / list / health
[ ] REST: POST /services, DELETE /services/{id}, GET /capabilities/{cap}
[ ] OpenAPI: auto-generiert

CLI
[ ] fs-registry list / lookup {capability} / status

Spezifisch
[ ] Wenn fertig: D24 (fs-bridge-sdk löschen) unblocked

Dokumentation
[ ] Doku-Seite: ServiceRegistry-Trait, Capabilities, API, CLI
[ ] commit + push
```

---

## fs-inventory (program)

> Was ist installiert?

```
OOP & Design
[ ] Repository Pattern
[ ] InventoryStore-Trait: upsert / uninstall / list / query / get
[ ] InstalledPackage: Status (Installing / Active / Failed / Removed)
[ ] Idempotent upsert
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Status-Texte + Fehlermeldungen (CLI + UI)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: upsert / uninstall / query / idempotenz getestet
[ ] cargo build --release: fehlerfrei

API
[ ] G8: Daemon vs. Bus-Subscriber (vor Umsetzung klären)
[ ] gRPC: list / upsert / uninstall / query / status
[ ] REST: GET /packages, POST /packages, DELETE /packages/{id}
[ ] OpenAPI: auto-generiert

CLI
[ ] fs-inventory list / status {id} / uninstall {id}

DB
[ ] Nur über fs-db DbEngine-Trait (fs-db-engine-sqlite default)
[ ] Migration-Datei anlegen

Dokumentation
[ ] Doku-Seite: InventoryStore-Trait, InstalledPackage, API, CLI, DB-Schema
[ ] commit + push
```

---

## fs-session (program)

> Wer ist eingeloggt? Welche Programme sind offen?

```
OOP & Design
[ ] Repository Pattern + Observer (SessionTracker)
[ ] SessionStore-Trait: open / close / minimize / restore / list / active-user
[ ] SessionTracker: beobachtet Fenster-Events vom Desktop
[ ] AppSession: Struct (app_id, state, opened_at)
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Fehlermeldungen (CLI + API)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: open / minimize / restore / close getestet
[ ] cargo build --release: fehlerfrei

API
[ ] G8: Daemon vs. Bus-Subscriber (vor Umsetzung klären)
[ ] gRPC: open / close / minimize / restore / list / active-user
[ ] REST: GET /sessions, POST /sessions, PUT /sessions/{id}
[ ] OpenAPI: auto-generiert

CLI
[ ] fs-session list / active / close {id}

DB
[ ] Nur über fs-db DbEngine-Trait
[ ] Migration-Datei anlegen

Dokumentation
[ ] Doku-Seite: SessionStore-Trait, AppSession, API, CLI, Desktop-Integration
[ ] commit + push
```

---

## fs-info (program)

> System-Info: CPU, RAM, Disk, Netzwerk, Uptime

```
OOP & Design
[ ] Facade Pattern: FsInfo fasst System-APIs zusammen
[ ] SystemInfo-Trait: cpu() / memory() / disk() / network() / uptime()
[ ] MetricsCollector-Trait: collect() → Vec<Metric>
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Einheiten + Labels + Fehlermeldungen (CLI + UI + TUI)
    "MB", "%", "CPU" etc. als FTL-Keys — auch CLI-Nutzer sehen sie
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

API
[ ] G8: Daemon vs. Library (vor Umsetzung klären)
[ ] gRPC: cpu / memory / disk / network / uptime / all
[ ] REST: GET /cpu, GET /memory, GET /disk, GET /uptime
[ ] OpenAPI: auto-generiert

UI
[ ] TUI-View: top-ähnliche Anzeige (fs-render CLI-Target)

CLI
[ ] fs-info cpu / memory / disk / all / uptime

Dokumentation
[ ] Doku-Seite: SystemInfo-Trait, Metriken, API, CLI, TUI
[ ] commit + push
```

---

## fs-container (program)

> Container-Abstraktion (Bollard / Podman)

```
OOP & Design
[ ] Adapter Pattern: ContainerAdapter wraps Bollard
[ ] ContainerEngine-Trait: start / stop / remove / list / logs / status / pull
[ ] ContainerSpec: Builder Pattern
[ ] Registriert Capability "container.engine" in fs-registry
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Status + Fehlermeldungen (CLI + UI + API)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC: start / stop / remove / list / logs / status / pull
[ ] REST: POST /containers, DELETE /containers/{id}, GET /containers/{id}/logs
[ ] OpenAPI: auto-generiert

CLI
[ ] fs-container list / start {id} / stop {id} / logs {id} / pull {image}

Dokumentation
[ ] Doku-Seite: ContainerEngine-Trait, ContainerAdapter, ContainerSpec, API, CLI
[ ] commit + push
```

---

## fs-federation (program)

> ActivityPub Federation

```
OOP & Design
[ ] Adapter Pattern: ActivityPubAdapter für ActivityPub-Protokoll
[ ] FederationGate-Trait: invite / accept / follow / unfollow / announce
[ ] Immer gegen Interface — fs-node kennt nur FederationGate-Trait

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys für alle user-facing Texte
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

Spezifisch (langfristig — nach G1)
[ ] P2: Federation-Grundstruktur
[ ] P3: Rechte-Kaskade + Audit-Log
[ ] P4: Föderaler Bus

Dokumentation
[ ] Doku-Seite: FederationGate-Trait, ActivityPub-Impl, Ablauf, API
[ ] commit + push
```

---

## fs-sync (program)

> CRDT-basierte Sync-Infrastruktur

```
OOP & Design
[ ] Strategy Pattern: SyncBackend austauschbar
[ ] SyncEngine-Trait: sync / status / resolve_conflicts / push / pull
[ ] CrdtStore-Trait: merge / apply / snapshot
[ ] ConflictResolver-Trait: resolve(local, remote) → Result
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Sync-Status + Fehlermeldungen (CLI + API)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: sync / merge / conflict-resolution getestet
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC: sync / status / conflicts / push / pull
[ ] REST: POST /sync, GET /status, GET /conflicts
[ ] OpenAPI: auto-generiert

CLI
[ ] fs-sync push / pull / status / conflicts

Dokumentation
[ ] Doku-Seite: SyncEngine-Trait, CRDT-Mechanismus, API, CLI
[ ] commit + push
```

---

## fs-channel (library, kein Container)

> Messenger-Abstraktion: definiert ChannelAdapter-Trait

```
OOP & Design
[ ] Strategy Pattern: ChannelAdapter je Messenger austauschbar
[ ] ChannelAdapter-Trait: send / receive / subscribe / list-rooms / join / leave
[ ] Immer gegen Interface — keine konkreten Messenger-Impl hier
[ ] Konkrete Impl: fs-channel-matrix, fs-channel-telegram (eigene Repos)

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Kein Containerfile (reine Trait-Library)

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] Keine FTL (reine Trait-Library)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: Trait-Definitionen + Stub-Impl getestet
[ ] cargo build --release: fehlerfrei

Dokumentation
[ ] Doku-Seite: ChannelAdapter-Trait, Wie Adapter-Repos den Trait implementieren
[ ] commit + push
```

---

## fs-llm (library, kein Container)

> LLM-Abstraktion: definiert LlmAdapter-Trait

```
OOP & Design
[ ] Strategy Pattern: LlmAdapter je Modell-Provider austauschbar
[ ] LlmAdapter-Trait: complete / embed / stream / list-models / health
[ ] ModelSelector: Strategy Pattern (welches Modell für welche Aufgabe)
[ ] Immer gegen Interface
[ ] Konkrete Impl: fs-llm-mistral, fs-llm-openai (eigene Repos)

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Kein Containerfile (reine Trait-Library)

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] Keine FTL (reine Trait-Library)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: Trait-Definitionen + Stub-Impl getestet
[ ] cargo build --release: fehlerfrei

Dokumentation
[ ] Doku-Seite: LlmAdapter-Trait, ModelSelector, Wie Adapter-Repos implementieren
[ ] commit + push
```

---

## fs-help (program)

> Context-sensitive Help Topics

```
OOP & Design
[ ] Repository Pattern
[ ] HelpProvider-Trait: get-topic / search / list-topics / related
[ ] HelpTopic: id + title + content-key (FTL-Key) + related + tags
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle Hilfetexte in locales/{lang}/help.ftl (CLI + UI + API)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC: get-topic / search / list-topics / related
[ ] REST: GET /topics, GET /topics/{id}, GET /search?q=
[ ] OpenAPI: auto-generiert

CLI
[ ] fs-help {topic} / search {query} / list

Dokumentation
[ ] Doku-Seite: HelpProvider-Trait, Topic-Format, FTL-Integration, API, CLI
[ ] commit + push
```

---

## fs-packages (program — Workspace: fs-pkg + fs-plugin-sdk + fs-plugin-runtime)

> Package-Typen, Install-Engine, Plugin-SDK, Plugin-Runtime

```
OOP & Design
[ ] Strategy Pattern: PackageInstaller je Paket-Typ
[ ] PackageInstaller-Trait: install / uninstall / validate / upgrade
[ ] ManifestParser-Trait: parse(bytes) → Manifest
[ ] PluginRuntime-Trait: load / unload / call / list (lädt Artifacts zur Laufzeit)
[ ] SetupStep-Trait: execute / rollback / validate — jeder Step ein Objekt
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Installer-Status + Fehlermeldungen (CLI + UI)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: Installer + Manifest-Parser + Plugin-Lifecycle getestet
[ ] cargo build --release: fehlerfrei

Spezifisch — große Dateien aufteilen
[ ] H4: manifest.rs (1394 Zeilen) → manifest/fields.rs + manifest/validation.rs + manifest/builder.rs
[ ] H6: setup_step.rs (886 Zeilen) → steps/download.rs + steps/verify.rs + steps/extract.rs + …
[ ] H7: installer.rs (706 Zeilen) → installer/prepare.rs + installer/execute.rs + installer/cleanup.rs

API
[ ] gRPC: install / uninstall / validate / upgrade / publish / load-artifact
[ ] REST: POST /install, DELETE /packages/{id}, POST /validate, POST /publish
[ ] OpenAPI: auto-generiert

CLI
[ ] fs-pkg install {id} / uninstall {id} / validate {manifest} / publish

Dokumentation
[ ] Doku-Seite: Manifest-Format, Installer-Phasen, Plugin-SDK-API, Artifact-Loading, CLI
[ ] commit + push
```

---

---

# Gruppe F — Programme

---

## fs-init (program)

> Installer + Erstkonfigurations-Programm

```
OOP & Design
[ ] State Machine: Installer-Phasen als States
[ ] InstallerStep-Trait: execute / rollback / display_name
[ ] BootcInstaller: Adapter für bootc install to-disk
[ ] VariantSelector: Server vs. Workstation (Strategy Pattern)
[ ] ConfigurationWizard: Chain of Responsibility
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] H12: Workspace-Cargo.toml im Root anlegen (cli/, validator/, migration/ haben eigene)
[ ] Containerfile (wird als Live-ISO-Payload ausgeführt)

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle Installer-Texte, Schritt-Beschreibungen, Fehlermeldungen (UI + CLI)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: alle InstallerStep-Impls getestet
[ ] cargo build --release: fehlerfrei

UI
[ ] Installer-Wizard: FsView-Trait (iced via fs-gui-engine-iced)
[ ] Varianten-Auswahl: Server / Workstation
[ ] Bundle-Auswahl: welche Pakete installieren?
[ ] Fortschrittsanzeige pro Schritt

CLI
[ ] fs-init install --variant server|workstation --disk /dev/sda
[ ] fs-init configure
[ ] fs-init status

Spezifisch
[ ] bootc install to-disk mit korrektem Image aufrufen
[ ] bootc switch für Remote-Migration
[ ] Store laden nach erstem Boot

Dokumentation
[ ] Doku-Seite: Installer-Ablauf, bootc-Integration, InstallerStep-Trait, Bundles, UI, CLI
[ ] commit + push
```

---

## fs-node (program)

> Orchestrator: AuthGateway, S3Provider, ServiceProxy, FederationGate, NodeAPI

```
OOP & Design
[ ] Facade Pattern: NodeServer fasst alle Schichten zusammen
[ ] NodeLayer-Trait: start / stop / name / health
[ ] AuthGateway: Adapter → fs-auth Protokoll-Traits
[ ] S3Provider: Adapter → fs-s3 (s3s + opendal)
[ ] ServiceProxy: Adapter → fs-registry
[ ] FederationGate: Adapter → fs-federation (deaktiviert bis Phase P)
[ ] Immer gegen Interface — nie Kanidm direkt aufrufen

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] H12: Workspace-Cargo.toml prüfen / anlegen
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Node-Status, Einladungs-Texte, Fehlermeldungen (CLI + UI + API)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: alle NodeLayer-Impls + Invite-System getestet
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC: status / layers / capabilities / invite-create / invite-accept
[ ] REST: GET /status, GET /capabilities, POST /invite, POST /invite/accept
[ ] OpenAPI: auto-generiert

CLI
[ ] fsn node serve
[ ] fsn node status
[ ] fsn node invite create / accept
[ ] fsn node capabilities

Invite-System
[ ] InviteToken: fsn1.… HMAC-signierter Token — Tests prüfen
[ ] InviteBundle: age-verschlüsseltes TOML — Tests prüfen
[ ] PortPool: dedizierter Port pro Einladung — Tests prüfen

Dokumentation
[ ] Doku-Seite: Schichten-Architektur, NodeLayer-Trait, Invite-System, API, CLI
[ ] commit + push
```

---

## fs-desktop (program)

> Wayland Shell: Workspace, Window-Management, Settings, Profile

```
OOP & Design
[ ] Elm-Pattern MVU: Message / Update / View (iced)
[ ] DesktopShell: zentrales Objekt
[ ] WindowManager: Observer Pattern (SessionTracker → Fenster-Events)
[ ] AppLauncher: Command Pattern
[ ] AppId-Enum: Strategy Pattern (welches App wird gerendert)
[ ] Settings-Module: je Modul eigenes Objekt
[ ] Domain-Objekte kein fs-render — nur view.rs als Bindeglied
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle Shell + Settings Texte (UI + CLI)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

Spezifisch — große Dateien aufteilen
[ ] H10a: fs-gui-workspace/src/desktop.rs (1362 Zeilen)
          → desktop/shell.rs + desktop/launcher.rs + desktop/workspace.rs + desktop/taskbar.rs
[ ] H10b: fs-settings/src/desktop_settings.rs (1172 Zeilen)
          → settings/display.rs + settings/input.rs + settings/power.rs + settings/workspace.rs
[ ] H10c: fs-settings/src/language.rs (1150 Zeilen)
          → settings/language/selector.rs + settings/language/download.rs + settings/language/preview.rs

G2.9 — Apps-Integration
[ ] Desktop zeigt echte Apps sobald jede App auf iced migriert ist
[ ] Placeholder entfernen wenn App fertig

CLI
[ ] fs-desktop status / list-windows / launch {app-id}

Langfristig
[ ] Wayland-Compositor (smithay/libcosmic) — fs-desktop als eigenständiger Compositor

Dokumentation
[ ] Doku-Seite: Shell-Architektur, Window-Management, Settings-Module, fs-render-Integration, CLI
[ ] commit + push
```

---

## fs-store (program)

> Paketmanager: Katalog, Suche, Installation — GUI wenn Display, sonst CLI

```
OOP & Design
[ ] Repository Pattern (CatalogRepository) + Facade (Store)
[ ] CatalogReader-Trait: search / list / get / info / check-updates
[ ] PackageInstaller-Trait: install / uninstall / update / validate
[ ] PackagePointer: Struct (id, sources-URLs, interfaces) — Pointer-Model
[ ] BundleLoader-Trait: load-bundle / list-bundles / install-bundle
[ ] Runtime-Erkennung: Display vorhanden → GUI, sonst CLI
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle Store-UI + CLI Texte
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: catalog-load + search + install-flow + bundle-load getestet
[ ] cargo build --release: fehlerfrei

Pointer-Model
[ ] package.toml lesen (id, name, version, description-en, sources, interfaces)
[ ] Beschreibungen aus Paket-Repo holen (nicht im Store-Katalog speichern)
[ ] Sprachdateien via OCI Artifact referenzieren
[ ] Bundles: bundle.toml aus Store/ Katalog lesen

API
[ ] gRPC: search / install / uninstall / update / list / info / install-bundle
[ ] REST: GET /packages, GET /packages/{id}, POST /packages/{id}/install, GET /bundles
[ ] OpenAPI: auto-generiert

UI
[ ] FsView-Trait (iced via fs-gui-engine-iced)
[ ] Paket-Suche + Details + Install-Button + Fortschritt
[ ] Bundle-Ansicht: welche Bundles gibt es?

CLI
[ ] fs-store search {query}
[ ] fs-store install {id}
[ ] fs-store install-bundle {bundle-id}
[ ] fs-store uninstall {id}
[ ] fs-store update [id]
[ ] fs-store list [--installed]
[ ] fs-store info {id}
[ ] fs-store bundles

Dokumentation
[ ] Doku-Seite: CatalogReader-Trait, Pointer-Model, Bundles, package.toml-Format, API, CLI
[ ] commit + push
```

---

## fs-icons (program)

> Icon-Sets: laden, suchen, nach Theme filtern

```
OOP & Design
[ ] Repository Pattern + Strategy (Theme-Filter)
[ ] IconProvider-Trait: get(id, size) / list() / search(query) / by-theme(id)
[ ] IconResolver: findet bestes Icon für Name + Größe + Theme
[ ] Immer gegen Interface

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Icon-Set-Namen + Beschreibungen + Fehlermeldungen (CLI + UI)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test
[ ] cargo build --release: fehlerfrei

API
[ ] gRPC: list / get / search / by-theme / list-sets
[ ] REST: GET /icons/{id}, GET /icons/search?q=, GET /sets
[ ] OpenAPI: auto-generiert

CLI
[ ] fs-icons list / get {id} / search {query} / by-theme {theme-id}

Dokumentation
[ ] Doku-Seite: IconProvider-Trait, IconSet-Format, Resolver, API, CLI
[ ] commit + push
```

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

Repo (in fs-apps/crates/fs-theme-app/)
[ ] CLAUDE.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle UI + CLI Texte
[ ] cargo clippy: 0 Fehler / cargo fmt / cargo test / cargo build --release

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

Repo (in fs-apps/crates/fs-lenses/)
[ ] CLAUDE.md / assets/icon.svg / package.toml / Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Lens-Namen + UI-Texte
[ ] cargo clippy: 0 Fehler / cargo fmt / cargo test / cargo build --release

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

Repo (in fs-apps/crates/fs-ai/)
[ ] CLAUDE.md / assets/icon.svg / package.toml / Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle UI + CLI Texte
[ ] cargo clippy: 0 Fehler / cargo fmt / cargo test / cargo build --release

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

Repo (in fs-apps/crates/fs-container-app/)
[ ] CLAUDE.md / assets/icon.svg / package.toml / Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle UI + CLI Texte
[ ] cargo clippy: 0 Fehler / cargo fmt / cargo test / cargo build --release

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

Repo (in fs-apps/crates/fs-tasks/)
[ ] CLAUDE.md / assets/icon.svg / package.toml / Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle UI + CLI Texte
[ ] cargo clippy: 0 Fehler / cargo fmt / cargo test / cargo build --release

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

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: Bot-Antworten + Fehlermeldungen (CLI + UI)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: BotAdapter-Trait + BotCommand getestet
[ ] cargo build --release: fehlerfrei

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

Repo (in fs-apps/crates/fs-builder/)
[ ] CLAUDE.md / assets/icon.svg / package.toml / Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle UI + CLI Texte
[ ] cargo clippy: 0 Fehler / cargo fmt / cargo test / cargo build --release

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

Repo
[ ] CLAUDE.md / rustfmt.toml / deny.toml / LICENSE / README.md / assets/icon.svg / package.toml
[ ] Containerfile

Code-Qualität
[ ] #![deny(clippy::all, clippy::pedantic, warnings)]
[ ] FTL-Keys: alle Manager-Texte (UI + CLI + API)
[ ] cargo clippy: 0 Fehler
[ ] cargo fmt --check: sauber
[ ] cargo test: alle Manager-Traits getestet
[ ] cargo build --release: fehlerfrei

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

## Store/ (Metadaten-Katalog — kein Container, kein Rust-Code)

> Nur TOML-Dateien: package.toml je Paket, bundle.toml je Bundle

```
[ ] package.toml Format festlegen:
    [package]   id, name, version, description-en, type (program|adapter|artifact|bundle)
    [sources]   readme-url, locales-url, binary.{arch}, container
    [interfaces] grpc-url, rest-url, openapi-url

[ ] bundle.toml Format festlegen:
    [bundle]    id, name, description-en
    [[packages]] id, version, optional (true/false)

[ ] Alle bestehenden Pakete auf neues Format migrieren
[ ] Neue Pakete eintragen: fs-bootc, fs-db-engine-sqlite, fs-db-engine-postgres,
    fs-llm-mistral, fs-llm-openai, fs-channel-matrix, fs-channel-telegram
[ ] Bundles anlegen: "server", "workstation", "minimal"
[ ] README.md: Katalog-Struktur erklären
[ ] commit + push
```

---

---

# Gruppe I — Dokumentation

---

## fs-documentation

```
[ ] INDEX.md aktualisieren

Neue Doku-Seiten:
[ ] Architektur: 3-Stufen-Installation (Init → Store → Bundles → Pakete → Artifacts)
[ ] Architektur: 4 Package-Typen (bundle / program / adapter / artifact)
[ ] Architektur: Artifact Global vs. Per-Paket
[ ] Architektur: Namenskonvention Adapter-Repos
[ ] Architektur: API-Standard (gRPC + REST + OpenAPI)
[ ] Architektur: DB-Engine-Strategie
[ ] Architektur: bootc OS-Images (fs-server + fs-workstation)
[ ] fs-bootc, fs-db-engine-sqlite, fs-db-engine-postgres
[ ] fs-llm-mistral, fs-llm-openai
[ ] fs-channel-matrix, fs-channel-telegram
[ ] fs-registry, fs-inventory, fs-session, fs-info
[ ] fs-channel, fs-llm (Trait-Libraries)
[ ] fs-sync, fs-federation, fs-help

[ ] commit + push
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
N0. [ ] PostgreSQL State Store (Blocker: fs-db-engine-postgres fertig)
        Bug: matrix feature → recursion overflow in rustc ≥1.94 (upstream)
```

---

## Reihenfolge

```
1.  Blocker klären (D24, H2)
2.  Architektur-Gespräche (G3, G7, G8)
3.  Neue Repos anlegen (fs-bootc, fs-db-engine-sqlite, fs-db-engine-postgres,
    fs-llm-mistral, fs-llm-openai, fs-channel-matrix, fs-channel-telegram)
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
