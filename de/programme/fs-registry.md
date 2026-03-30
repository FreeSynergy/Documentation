# fs-registry

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-registry` verwaltet, welche Services und Capabilities gerade auf einem Node laufen.
Andere Services registrieren sich beim Start und melden sich beim Herunterfahren ab.
Consumer fragen die Registry, um den Endpoint für eine gewünschte Capability zu finden —
ohne hart kodierte Adressen.

---

## Architektur

```
ServiceRegistry-Trait  ← Registry Pattern (Strategy)
       ↑
   Registry            ← konkrete SQLite-Impl (Arc<dyn ServiceRegistry> überall)
       ↑
GrpcServer  RestServer  CLI       BusHandler
```

Kein Consumer kennt die konkrete `Registry`-Implementierung — nur `dyn ServiceRegistry`.

---

## ServiceRegistry-Trait

| Methode            | Beschreibung                                             |
|--------------------|----------------------------------------------------------|
| `register`         | Service-Capability-Eintrag registrieren (idempotent)     |
| `deregister`       | Alle Capabilities einer Service-ID abmelden              |
| `list`             | Alle registrierten Einträge abfragen                     |
| `by_capability`    | Alle Services für eine Capability-ID                     |
| `by_service`       | Alle Capabilities einer Service-ID                       |
| `endpoint_for`     | Endpoint für eine spezifische Capability                 |
| `set_status`       | Status eines Eintrags setzen (Running / Stopped / Error) |
| `health`           | Health-Check des Registry-Dienstes selbst                |

---

## API

### gRPC (Port 50060)

Proto-Datei: `proto/registry.proto`

| RPC              | Beschreibung                                      |
|------------------|---------------------------------------------------|
| `Register`       | Service + Capability registrieren                 |
| `Deregister`     | Service abmelden                                  |
| `List`           | Alle Einträge abfragen                            |
| `LookupCapability` | Services für eine Capability suchen             |
| `EndpointFor`    | Endpoint für eine Capability                      |
| `SetStatus`      | Status eines Eintrags setzen                      |
| `Health`         | Health-Probe                                      |

### REST (Port 8060)

| Methode  | Pfad                               | Beschreibung                        |
|----------|------------------------------------|-------------------------------------|
| `POST`   | `/services`                        | Service registrieren                |
| `DELETE` | `/services/{id}`                   | Service abmelden                    |
| `GET`    | `/services`                        | Alle Services                       |
| `GET`    | `/capabilities/{id}`               | Services für eine Capability        |
| `GET`    | `/endpoint/{capability}`           | Endpoint für eine Capability        |
| `PUT`    | `/services/{id}/status`            | Status setzen                       |
| `GET`    | `/health`                          | Health-Probe                        |
| `GET`    | `/swagger-ui`                      | OpenAPI-Dokumentation               |

---

## CLI

```
fs-registry list                    — Alle registrierten Services anzeigen
fs-registry capability {id}         — Services für eine Capability
fs-registry status                  — Health-Probe
```

---

## Konfiguration (Umgebungsvariablen)

| Variable         | Standard                              | Beschreibung                    |
|------------------|---------------------------------------|---------------------------------|
| `FS_REGISTRY_DB` | `/var/lib/freesynergy/registry.db`    | Pfad zur SQLite-Datenbank       |
| `FS_GRPC_PORT`   | `50060`                               | gRPC-Port                       |
| `FS_REST_PORT`   | `8060`                                | REST-Port                       |

---

## Wie andere Services sich registrieren

```rust
use fs_registry::{Registry, ServiceEntry};

let registry = Registry::open(&db_path).await?;
let entry = ServiceEntry::new("fs-db-engine-sqlite", "db.engine.sqlite", "http://0.0.0.0:8050");
registry.register(entry).await?;
```

---

## Repo

- Lokal: `/home/kal/Server/fs-registry/`
- GitHub: `git@github.com:FreeSynergy/fs-registry.git`

---

Weiter: [fs-inventory](fs-inventory.md) | [← Index](../INDEX.md)
