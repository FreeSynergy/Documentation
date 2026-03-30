# fs-channel-telegram

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-channel-telegram` ist der Telegram-Channel-Adapter für FreeSynergy.
Er implementiert den `Channel`-Trait aus `fs-channel` (Feature `telegram`) via `teloxide`.
Läuft als eigenständiger Microservice mit gRPC- und REST-Endpunkt.
Registriert die Capability `channel.telegram` in `fs-registry`.

---

## Architektur

```
TelegramAdapter  ← implementiert Channel-Trait (fs-channel, Feature telegram)
     ↑
GrpcChannel      ← tonic-Service (TelegramAdapter hinter Arc)
REST-Router      ← axum-Routen  (TelegramAdapter hinter Arc)
```

**Adapter Pattern**: Wraps `TelegramAdapter` aus `fs-channel` (Feature `telegram`).

> **Hinweis**: Streaming-Subscribe (gRPC) gibt `unimplemented` zurück — Telegram Bot API
> unterstützt kein Server-Push. Für Nachrichten-Empfang Long-Polling via REST verwenden.

---

## Capability

| Feld           | Wert                        |
|----------------|-----------------------------|
| ID             | `channel.telegram`          |
| Registriert in | `fs-registry`               |
| Endpunkt       | `http://0.0.0.0:8072`       |

---

## API

### gRPC (Port 50072)

Proto-Datei: `proto/channel.proto` (geteilt mit `fs-channel-matrix`)

| RPC         | Beschreibung                                          |
|-------------|-------------------------------------------------------|
| `Send`      | Nachricht an einen Chat senden                        |
| `Subscribe` | Stream eingehender Nachrichten (unimplemented)        |
| `ListRooms` | Verfügbare Chats abfragen                             |
| `Join`      | Chat beitreten                                        |
| `Leave`     | Chat verlassen                                        |
| `Health`    | Erreichbarkeit + Verbindungsstatus                   |

### REST (Port 8072)

| Methode | Pfad         | Beschreibung                          |
|---------|--------------|---------------------------------------|
| `GET`   | `/health`    | Health-Probe                          |
| `POST`  | `/send`      | Nachricht senden                      |
| `GET`   | `/messages`  | Nachrichten abrufen (Long-Polling)    |
| `GET`   | `/swagger-ui`| OpenAPI-Dokumentation                 |

---

## Konfiguration (Umgebungsvariablen)

| Variable                | Standard                              | Beschreibung                  |
|-------------------------|---------------------------------------|-------------------------------|
| `FS_TELEGRAM_BOT_TOKEN` | (erforderlich)                        | Telegram Bot Token            |
| `FS_GRPC_PORT`          | `50072`                               | gRPC-Port                     |
| `FS_REST_PORT`          | `8072`                                | REST-Port                     |
| `FS_REGISTRY_DB`        | `/var/lib/freesynergy/registry.db`    | Pfad zur Registry-Datenbank   |
| `FS_LOCALES_DIR`        | `/usr/share/freesynergy/locales`      | Verzeichnis der .ftl-Dateien  |

---

## i18n

FTL-Datei: `fs-i18n/locales/{lang}/channel-telegram.ftl`

---

## Repo

- Lokal: `/home/kal/Server/fs-channel-telegram/`
- GitHub: `git@github.com:FreeSynergy/fs-channel-telegram.git`

---

Weiter: [← Index](../INDEX.md)
