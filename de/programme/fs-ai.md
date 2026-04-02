# fs-ai

**AI-Assistent für FreeSynergy**

fs-ai implementiert das **Facade Pattern**: Der `AiController` ist eine Fassade über `fs-manager-ai` und kapselt die LLM-Engine-Verwaltung (starten, stoppen, Status).

## Architektur

```
AiController (Facade über fs-manager-ai)
├── AiView (FsView-Trait, view.rs)
├── GrpcAiApp (gRPC: list/status/start/stop/health)
├── REST + OpenAPI (axum + utoipa)
└── CLI (fs-ai)
```

## Unterstützte Engines

| Engine | Beschreibung |
|--------|-------------|
| Mistral.rs | Hochperformante LLM-Inferenz, OpenAI-kompatible API |

## Verzeichnis

| Pfad | Inhalt |
|------|--------|
| `src/controller.rs` | AiController (Facade Pattern) |
| `src/model.rs` | AiModel, KnownModel |
| `src/view.rs` | AiView (FsView) |
| `src/grpc.rs` | gRPC service |
| `src/rest.rs` | REST + OpenAPI |
| `src/cli.rs` | CLI |
| `src/keys.rs` | FTL-Schlüssel-Konstanten |
| `proto/ai.proto` | gRPC Protokolldefinition |

## FTL-Strings

Alle UI-Texte in `fs-i18n/locales/{lang}/ai.ftl`.

## Repo

`git@github.com:FreeSynergy/fs-ai.git`
