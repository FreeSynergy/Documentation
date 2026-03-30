# fs-llm-openai

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-llm-openai` ist der OpenAI-LLM-Adapter für FreeSynergy.
Er implementiert den `LlmProvider`-Trait aus `fs-llm` und leitet Anfragen an die
OpenAI-API weiter. Läuft als eigenständiger Microservice mit gRPC- und REST-Endpunkt.

---

## Architektur

```
OpenAiAdapter  ← implementiert LlmProvider-Trait (fs-llm)
     ↑
GrpcProvider   ← tonic-Service (OpenAiAdapter hinter Arc)
REST-Router    ← axum-Routen  (OpenAiAdapter hinter Arc)
```

**Adapter Pattern**: Wraps `OpenAiCompatProvider` aus `fs-llm` gegen `api.openai.com`.

---

## Capability

| Feld           | Wert                        |
|----------------|-----------------------------|
| ID             | `llm.openai`                |
| Registriert in | `fs-registry`               |
| Endpunkt       | `http://0.0.0.0:8062`       |

---

## API

### gRPC (Port 50062)

Proto-Datei: `proto/llm_provider.proto` (geteilt mit `fs-llm-mistral`)

| RPC          | Beschreibung                        |
|--------------|-------------------------------------|
| `Complete`   | Completion für Nachrichten-Liste    |
| `Embed`      | Text einbetten → Float-Vektor       |
| `ListModels` | Verfügbare OpenAI-Modelle abfragen  |
| `Health`     | API-Erreichbarkeit + Latenz         |

### REST (Port 8062)

| Methode | Pfad         | Beschreibung                          |
|---------|--------------|---------------------------------------|
| `GET`   | `/health`    | Health-Probe: Erreichbarkeit + Latenz |
| `POST`  | `/complete`  | Completion (`{"messages":[...]}`)     |
| `POST`  | `/embed`     | Embedding (`{"input":"..."}`)         |
| `GET`   | `/models`    | Modell-Liste von OpenAI               |
| `GET`   | `/swagger-ui`| OpenAPI-Dokumentation                 |

---

## Konfiguration (Umgebungsvariablen)

| Variable             | Standard                              | Beschreibung                |
|----------------------|---------------------------------------|-----------------------------|
| `FS_OPENAI_API_KEY`  | (Pflichtfeld)                         | OpenAI-API-Schlüssel        |
| `FS_OPENAI_MODEL`    | `gpt-4o`                              | Standardmodell              |
| `FS_GRPC_PORT`       | `50062`                               | gRPC-Port                   |
| `FS_REST_PORT`       | `8062`                                | REST-Port                   |
| `FS_REGISTRY_DB`     | `/var/lib/freesynergy/registry.db`    | Pfad zur Registry-Datenbank |
| `FS_LOCALES_DIR`     | `/usr/share/freesynergy/locales`      | Verzeichnis der .ftl-Dateien|

---

## i18n

FTL-Datei: `fs-i18n/locales/{lang}/llm-openai.ftl`

---

## Repo

- Lokal: `/home/kal/Server/fs-llm-openai/`
- GitHub: `git@github.com:FreeSynergy/fs-llm-openai.git`

---

Weiter: [fs-llm-mistral](fs-llm-mistral.md) | [AI-Inferenz](../konzepte/ai-inferenz.md)
