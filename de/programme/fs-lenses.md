# fs-lenses

**Aggregierte Daten-Views für FreeSynergy**

fs-lenses implementiert das **Strategy Pattern** für kontextabhängige Suchergebnisse: je nach Kontext wird die passende Lens-Strategie gewählt (Wiki, Git, Chat, Aufgaben).

## Architektur

```
LensController (Strategy Pattern)
├── LensesView / LensDetailView (FsView-Trait, view.rs)
├── GrpcLensApp (gRPC: list/create/delete/query/health)
├── REST + OpenAPI (axum + utoipa)
└── CLI (fs-lenses)
```

## Lens-Typen

| Lens | Beschreibung |
|------|-------------|
| Wiki | Suchergebnisse aus dem Wiki |
| Git | Passende Git-Repositories |
| Chat | Nachrichten mit dem Suchbegriff |
| Tasks | Offene Aufgaben |

## Verzeichnis

| Pfad | Inhalt |
|------|--------|
| `src/controller.rs` | LensController (Strategy Pattern) |
| `src/model.rs` | Lens, LensRole |
| `src/query.rs` | LensQueryEngine-Trait |
| `src/view.rs` | LensesView, LensDetailView (FsView) |
| `src/grpc.rs` | gRPC service |
| `src/rest.rs` | REST + OpenAPI |
| `src/cli.rs` | CLI |
| `src/keys.rs` | FTL-Schlüssel-Konstanten |
| `proto/lenses.proto` | gRPC Protokolldefinition |

## FTL-Strings

Alle UI-Texte in `fs-i18n/locales/{lang}/lenses.ftl`.

## Repo

`git@github.com:FreeSynergy/fs-lenses.git`
