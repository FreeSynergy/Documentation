# fs-db-engine-sqlite

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-db-engine-sqlite` ist der SQLite-Adapter für die FreeSynergy-Datenbankabstraktion.
Er implementiert den `DbEngine`-Trait aus `fs-db` und läuft als eigenständiger Microservice
mit gRPC- und REST-Endpunkt.

---

## Architektur

```
SqliteEngine  ← implementiert DbEngine-Trait (fs-db)
     ↑
GrpcEngine    ← tonic-Service (SqliteEngine hinter RwLock)
REST-Router   ← axum-Routen   (SqliteEngine hinter RwLock)
```

**Adapter Pattern**: Consumer kennen nur den `DbEngine`-Trait — nie diese Crate direkt.

---

## Capability

| Feld       | Wert                       |
|------------|----------------------------|
| ID         | `db.engine.sqlite`         |
| Registriert in | `fs-registry`          |
| Endpunkt   | `http://0.0.0.0:8081`      |

Beim Start registriert der Service seine Capability in `fs-registry`.
Andere Services fragen dort ab, welche DB-Engines verfügbar sind.

---

## API

### gRPC (Port 50051)

Proto-Datei: `proto/db_engine.proto` (geteilt mit `fs-db-engine-postgres`)

| RPC       | Beschreibung                          |
|-----------|---------------------------------------|
| `Open`    | Verbindung öffnen (URL + max. Pools)  |
| `Close`   | Verbindung schließen                  |
| `Migrate` | DDL-Migrationen ausführen             |
| `Execute` | SQL ausführen (SELECT oder DML)       |
| `Health`  | Verbindungsstatus + SQLite-Version    |

### REST (Port 8081)

| Methode | Pfad         | Beschreibung                        |
|---------|--------------|-------------------------------------|
| `GET`   | `/health`    | Health-Probe: Status + Version      |
| `POST`  | `/migrate`   | Migrationen ausführen               |
| `POST`  | `/execute`   | SQL ausführen (`{"sql":"...","params":[...]}`) |
| `GET`   | `/swagger-ui`| OpenAPI-Dokumentation               |

---

## Konfiguration (Umgebungsvariablen)

| Variable         | Standard                              | Beschreibung               |
|------------------|---------------------------------------|----------------------------|
| `FS_DB_PATH`     | `/var/lib/freesynergy/sqlite.db`      | SQLite-Datenbankpfad       |
| `FS_GRPC_PORT`   | `50051`                               | gRPC-Port                  |
| `FS_REST_PORT`   | `8081`                                | REST-Port                  |
| `FS_REGISTRY_DB` | `/var/lib/freesynergy/registry.db`    | Pfad zur Registry-Datenbank|
| `FS_LOCALES_DIR` | `/usr/share/freesynergy/locales`      | Verzeichnis der .ftl-Dateien|

---

## i18n

Alle user-facing Fehlermeldungen kommen aus:
`fs-i18n/locales/{lang}/db-engine-sqlite.ftl`

Verwendete FTL-Keys (Auszug):

| Key                          | Beschreibung                        |
|------------------------------|-------------------------------------|
| `db-sqlite-capability-name`  | Anzeigename der Capability          |
| `db-sqlite-error-connect`    | Verbindungsfehler (var: `$url`)     |
| `db-sqlite-error-query`      | Abfragefehler (var: `$reason`)      |
| `db-sqlite-error-health`     | Health-Check-Fehler (var: `$reason`)|

---

## Repo

- Lokal: `/home/kal/Server/fs-db-engine-sqlite/`
- GitHub: `git@github.com:FreeSynergy/fs-db-engine-sqlite.git`

---

Weiter: [Datenspeicherung](datenspeicherung.md) | [Registry-Konzept](../konzepte/registry.md)
