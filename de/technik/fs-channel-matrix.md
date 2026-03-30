# fs-channel-matrix

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-channel-matrix` ist der Matrix-Channel-Adapter für FreeSynergy.
Er implementiert den `Channel`-Trait aus `fs-channel` via `matrix-sdk`.
Läuft als eigenständiger Microservice mit gRPC- und REST-Endpunkt.
Registriert die Capability `channel.matrix` in `fs-registry`.

> **Bekanntes Problem**: `matrix-sdk 0.16` verursacht einen Rekursions-Overflow in `rustc ≥ 1.94`.
> Der Build mit `--features live` ist blockiert bis upstream einen Fix liefert.
> Der Default-Build (ohne `live`) läuft sauber — alle Channel-Operationen geben "not available" zurück.

---

## Architektur

```
MatrixAdapter   ← implementiert Channel-Trait (fs-channel) [nur mit Feature live]
     ↑
GrpcChannel     ← tonic-Service (stub ohne live, live-Impl mit Feature live)
REST-Router     ← axum-Routen  (stub ohne live)
```

**Feature-Flag**: `live` aktiviert `matrix-sdk` und echte Matrix-Konnektivität.

---

## Capability

| Feld           | Wert                        |
|----------------|-----------------------------|
| ID             | `channel.matrix`            |
| Registriert in | `fs-registry`               |
| Endpunkt       | `http://0.0.0.0:8071`       |

---

## API

### gRPC (Port 50071)

Proto-Datei: `proto/channel.proto` (geteilt mit `fs-channel-telegram`)

| RPC         | Beschreibung                                 |
|-------------|----------------------------------------------|
| `Send`      | Nachricht in einen Room senden               |
| `Subscribe` | Stream eingehender Nachrichten (Server-Push) |
| `ListRooms` | Verfügbare Rooms abfragen                    |
| `Join`      | Room beitreten                               |
| `Leave`     | Room verlassen                               |
| `Health`    | Erreichbarkeit + Verbindungsstatus           |

### REST (Port 8071)

| Methode | Pfad         | Beschreibung                          |
|---------|--------------|---------------------------------------|
| `GET`   | `/health`    | Health-Probe                          |
| `POST`  | `/send`      | Nachricht senden                      |
| `GET`   | `/rooms`     | Room-Liste                            |
| `GET`   | `/swagger-ui`| OpenAPI-Dokumentation                 |

---

## Konfiguration (Umgebungsvariablen)

| Variable          | Standard                              | Beschreibung                  |
|-------------------|---------------------------------------|-------------------------------|
| `FS_MATRIX_URL`   | (erforderlich)                        | URL des Matrix-Homeservers    |
| `FS_MATRIX_USER`  | (erforderlich)                        | Matrix-Benutzername           |
| `FS_MATRIX_TOKEN` | (oder FS_MATRIX_PASSWORD)             | Access Token                  |
| `FS_GRPC_PORT`    | `50071`                               | gRPC-Port                     |
| `FS_REST_PORT`    | `8071`                                | REST-Port                     |
| `FS_REGISTRY_DB`  | `/var/lib/freesynergy/registry.db`    | Pfad zur Registry-Datenbank   |
| `FS_LOCALES_DIR`  | `/usr/share/freesynergy/locales`      | Verzeichnis der .ftl-Dateien  |

---

## i18n

FTL-Datei: `fs-i18n/locales/{lang}/channel-matrix.ftl`

---

## Repo

- Lokal: `/home/kal/Server/fs-channel-matrix/`
- GitHub: `git@github.com:FreeSynergy/fs-channel-matrix.git`

---

Weiter: [fs-channel-telegram](fs-channel-telegram.md)
