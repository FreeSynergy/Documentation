# fs-db-engine-postgres

**PostgreSQL-Adapter für FreeSynergy**

Implementiert den `DbEngine`-Trait aus `fs-db` auf Basis von `sqlx` + PostgreSQL.
Stellt sich als gRPC- + REST-Microservice zur Verfügung und registriert die
Capability `db.engine.postgres` beim Start in `fs-registry`.

---

## Architektur

```
fs-db (DbEngine-Trait — keine Engine-Abhängigkeit)
    └── fs-db-engine-postgres
            PostgresEngine  → implementiert DbEngine
            GrpcEngine      → tonic DbEngineService (eigene RwLock-Verbindung)
            REST router     → axum /health, /migrate, /execute
            capability      → "db.engine.postgres" für fs-registry
```

**Adapter Pattern**: `PostgresEngine` adaptiert die sqlx-PgPool-API an die
`DbEngine`-Schnittstelle.  Consumer-Code importiert nur `fs-db` — die konkrete
Engine wird über Dependency Injection oder Environment-Variable gewählt.

---

## Module

### `PostgresEngine` — `DbEngine`-Implementierung

| Methode | Beschreibung |
|---|---|
| `open(config)` | Verbindet mit PostgreSQL via `PgPoolOptions` |
| `migrate()` | No-op — Consumer übergeben DDL über `execute()` |
| `execute(sql, params)` | SELECT/WITH → `query()`, sonst → `dml()` |
| `health()` | `SELECT version()` mit Latenz-Messung |
| `close()` | Schließt den Connection-Pool |

Parameter werden als `Vec<serde_json::Value>` übergeben und via `bind_json()`
an die sqlx-Query gebunden (NULL, bool, i64, f64, String).  Spalten werden
in `DbRow { columns, values }` konvertiert; Typ-Mapping: INT\*/BOOL → native
JSON-Typen, alles andere → String-Fallback.

### `GrpcEngine` — tonic `DbEngineService`

Eigenständige Verbindung (kein Shared-State mit REST).  Clients können via
`Open`-RPC eine eigene Verbindung öffnen und via `Close`-RPC schließen.

| RPC | Beschreibung |
|---|---|
| `Open(url, max_connections)` | Öffnet PostgresEngine |
| `Close()` | Schließt Engine + Pool |
| `Migrate()` | Führt Migrationen aus |
| `Execute(sql, params)` | Führt SQL aus, gibt Rows zurück |
| `Health()` | Gibt Verbindungsstatus + Version zurück |

Proto-Definition: `proto/db_engine.proto` (geteilt mit `fs-db-engine-sqlite`).

### REST-Router (axum)

| Route | Methode | Beschreibung |
|---|---|---|
| `/health` | GET | Verbindungsstatus + PostgreSQL-Version |
| `/migrate` | POST | Migrationen ausführen |
| `/execute` | POST | SQL ausführen (`{ "sql": "...", "params": [...] }`) |
| `/swagger-ui` | GET | OpenAPI-Dokumentation (utoipa) |

### `capability` — `db.engine.postgres`

Stabiler Capability-String: `"db.engine.postgres"`.

Format: `<domain>.<type>.<impl>`
- `db` — Datenbank-Subsystem
- `engine` — Markiert dies als `DbEngine`-Implementierung
- `postgres` — Konkreter Backend-Name

`PostgresCapability::descriptor()` liefert ID, lokalisierten Anzeigenamen
(FTL-Key `db-engine-postgres-name`) und Crate-Version.

---

## Capability-Registration

Beim Start öffnet `main.rs` die `fs-registry`-Datenbank und registriert:

```
service_id  = "fs-db-engine-postgres"
capability  = "db.engine.postgres"
endpoint    = "http://<FS_REST_ADDR>"
```

Bei sauberem Shutdown (nach `tokio::select!`) wird die Capability wieder
deregistriert.  Fehler beim Deregistrieren werden als Warning geloggt —
der Service-Start schlägt nicht fehl.

---

## Konfiguration (Environment-Variablen)

| Variable | Default | Beschreibung |
|---|---|---|
| `FS_DB_URL` | `postgres://freesynergy:freesynergy@localhost:5432/freesynergy` | PostgreSQL-URL |
| `FS_GRPC_PORT` | `50052` | gRPC-Port |
| `FS_REST_PORT` | `8082` | REST-Port |
| `FS_REGISTRY_PATH` | `/var/lib/freesynergy/registry.db` | Pfad zur fs-registry SQLite-DB |
| `FS_LANG` | `en` | Aktive Sprache für i18n |

---

## i18n

Alle user-facing Fehlermeldungen nutzen FTL-Keys aus `fs-i18n/locales/{lang}/db.ftl`:

| Key | Variable | Beschreibung |
|---|---|---|
| `db-error-connection-failed` | `$url` | Verbindungsfehler |
| `db-error-query-failed` | `$reason` | Query-/DML-Fehler |
| `db-error-health-failed` | `$reason` | Health-Check-Fehler |
| `db-engine-postgres-name` | — | Anzeigename ("PostgreSQL") |

---

## Tests

Integration-Tests benötigen eine laufende PostgreSQL-Instanz:

```bash
FS_TEST_PG_URL=postgres://user:pass@localhost/test cargo test
```

Ohne `FS_TEST_PG_URL` werden die Tests übersprungen (kein Fehler).

---

## Ports

| Server | Default |
|---|---|
| gRPC | 50052 |
| REST | 8082 |

---

## Abhängigkeiten

| Crate | Zweck |
|---|---|
| `fs-db` | `DbEngine`-Trait + `DbConfig`, `DbRow`, `DbRows`, `DbHealth` |
| `fs-error` | `FsError` |
| `fs-i18n` | Lokalisierte Fehlermeldungen |
| `fs-registry` | Capability-Registration |
| `sqlx 0.8` (postgres) | Connection-Pool + Query-Execution |
| `tonic 0.12` | gRPC-Server |
| `axum 0.7` | REST-Server |
| `utoipa 5` | OpenAPI-Dokumentation |
