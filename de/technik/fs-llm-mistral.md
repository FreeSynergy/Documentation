# fs-llm-mistral

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-llm-mistral` ist der Mistral-LLM-Adapter für FreeSynergy.
Er implementiert den `LlmProvider`-Trait aus `fs-llm` und leitet Anfragen an einen
laufenden `mistral.rs`-Server weiter (OpenAI-kompatible REST-API).
Läuft als eigenständiger Microservice mit gRPC- und REST-Endpunkt.

---

## Architektur

```
MistralAdapter  ← implementiert LlmProvider-Trait (fs-llm)
     ↑
GrpcProvider    ← tonic-Service (MistralAdapter hinter Arc)
REST-Router     ← axum-Routen  (MistralAdapter hinter Arc)
```

**Adapter Pattern**: Wraps `OpenAiCompatProvider` aus `fs-llm` gegen den mistral.rs-Server.

---

## Capability

| Feld           | Wert                        |
|----------------|-----------------------------|
| ID             | `llm.mistral`               |
| Registriert in | `fs-registry`               |
| Endpunkt       | `http://0.0.0.0:8061`       |

---

## API

### gRPC (Port 50061)

Proto-Datei: `proto/llm_provider.proto` (geteilt mit `fs-llm-openai`)

| RPC          | Beschreibung                               |
|--------------|--------------------------------------------|
| `Complete`   | Completion für Nachrichten-Liste            |
| `Embed`      | Text einbetten → Float-Vektor              |
| `ListModels` | Verfügbare Modelle auf dem Server abfragen |
| `Health`     | Erreichbarkeit + Latenz                    |

### REST (Port 8061)

| Methode | Pfad         | Beschreibung                          |
|---------|--------------|---------------------------------------|
| `GET`   | `/health`    | Health-Probe: Erreichbarkeit + Latenz |
| `POST`  | `/complete`  | Completion (`{"messages":[...]}`)     |
| `POST`  | `/embed`     | Embedding (`{"input":"..."}`)         |
| `GET`   | `/models`    | Modell-Liste vom mistral.rs-Server    |
| `GET`   | `/swagger-ui`| OpenAPI-Dokumentation                 |

---

## Konfiguration (Umgebungsvariablen)

| Variable           | Standard                              | Beschreibung                  |
|--------------------|---------------------------------------|-------------------------------|
| `FS_MISTRAL_URL`   | `http://localhost:1234`               | URL des mistral.rs-Servers    |
| `FS_MISTRAL_MODEL` | `mistral-7b-instruct`                 | Standardmodell                |
| `FS_GRPC_PORT`     | `50061`                               | gRPC-Port                     |
| `FS_REST_PORT`     | `8061`                                | REST-Port                     |
| `FS_REGISTRY_DB`   | `/var/lib/freesynergy/registry.db`    | Pfad zur Registry-Datenbank   |
| `FS_LOCALES_DIR`   | `/usr/share/freesynergy/locales`      | Verzeichnis der .ftl-Dateien  |

---

## i18n

FTL-Datei: `fs-i18n/locales/{lang}/llm-mistral.ftl`

---

## Repo

- Lokal: `/home/kal/Server/fs-llm-mistral/`
- GitHub: `git@github.com:FreeSynergy/fs-llm-mistral.git`

---

Weiter: [fs-llm-openai](fs-llm-openai.md) | [AI-Inferenz](../konzepte/ai-inferenz.md)
