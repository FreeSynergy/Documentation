# fs-db-engine-postgres

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-db-engine-postgres` ist der PostgreSQL-Adapter für die FreeSynergy-Datenbankabstraktion.
Er implementiert den `DbEngine`-Trait aus `fs-db` und läuft als eigenständiger Microservice
mit gRPC- und REST-Endpunkt.

---

## Architektur

```
PostgresEngine  ← implementiert DbEngine-Trait (fs-db)
     ↑
GrpcEngine      ← tonic-Service (PostgresEngine hinter RwLock)
REST-Router     ← axum-Routen   (PostgresEngine hinter RwLock)
```

**Adapter Pattern**: Consumer kennen nur den `DbEngine`-Trait — nie diese Crate direkt.

---

## Capability

| Feld           | Wert                        |
|----------------|-----------------------------|
| ID             | `db.engine.postgres`        |
| Registriert in | `fs-registry`               |
| Endpunkt       | `http://0.0.0.0:8082`       |

Beim Start registriert der Service seine Capability in `fs-registry`.

---

## API

### gRPC (Port 50052)

Proto-Datei: `proto/db_engine.proto` (geteilt mit `fs-db-engine-sqlite`)

| RPC       | Beschreibung                          |
|-----------|---------------------------------------|
| `Open`    | Verbindung öffnen (URL + max. Pools)  |
| `Close`   | Verbindung schließen                  |
| `Migrate` | DDL-Migrationen ausführen             |
| `Execute` | SQL ausführen (SELECT oder DML)       |
| `Health`  | Verbindungsstatus + PostgreSQL-Version|

### REST (Port 8082)

| Methode | Pfad         | Beschreibung                        |
|---------|--------------|-------------------------------------|
| `GET`   | `/health`    | Health-Probe: Status + Version      |
| `POST`  | `/migrate`   | Migrationen ausführen               |
| `POST`  | `/execute`   | SQL ausführen (`{"sql":"...","params":[...]}`) |
| `GET`   | `/swagger-ui`| OpenAPI-Dokumentation               |

---

## Konfiguration (Umgebungsvariablen)

| Variable           | Standard                              | Beschreibung                |
|--------------------|---------------------------------------|-----------------------------|
| `FS_DB_URL`        | `postgres://localhost/freesynergy`    | PostgreSQL Connection URL   |
| `FS_GRPC_PORT`     | `50052`                               | gRPC-Port                   |
| `FS_REST_PORT`     | `8082`                                | REST-Port                   |
| `FS_REGISTRY_DB`   | `/var/lib/freesynergy/registry.db`    | Pfad zur Registry-Datenbank |
| `FS_LOCALES_DIR`   | `/usr/share/freesynergy/locales`      | Verzeichnis der .ftl-Dateien|

Integration Tests benötigen `FS_TEST_PG_URL` (werden sonst übersprungen).

---

## i18n

Alle user-facing Fehlermeldungen kommen aus:
`fs-i18n/locales/{lang}/db-engine-postgres.ftl`

Verwendete FTL-Keys (Auszug):

| Key                              | Beschreibung                        |
|----------------------------------|-------------------------------------|
| `db-postgres-capability-name`    | Anzeigename der Capability          |
| `db-postgres-error-connect`      | Verbindungsfehler (var: `$url`)     |
| `db-postgres-error-query`        | Abfragefehler (var: `$reason`)      |
| `db-postgres-error-health`       | Health-Check-Fehler (var: `$reason`)|

---

## Repo

- Lokal: `/home/kal/Server/fs-db-engine-postgres/`
- GitHub: `git@github.com:FreeSynergy/fs-db-engine-postgres.git`

---

Weiter: [fs-db-engine-sqlite](fs-db-engine-sqlite.md) | [Datenspeicherung](datenspeicherung.md)
