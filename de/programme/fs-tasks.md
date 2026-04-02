# fs-tasks

**Task-Pipeline-Manager für FreeSynergy**

fs-tasks implementiert das **Command Pattern**: Jede Pipeline ist ein ausführbarer Befehl mit konfigurierbaren Trigger, Quelle (Source) und Ziel (Target).

## Architektur

```
TaskController (Command Pattern)
├── TasksView / TaskDetailView / CreateTaskView (FsView-Trait, view.rs)
├── GrpcTasksApp (gRPC: list/create/delete/toggle/health)
├── REST + OpenAPI (axum + utoipa)
└── CLI (fs-tasks)
```

## Datenmodell

| Typ | Beschreibung |
|-----|-------------|
| `TaskPipeline` | Eine Automatisierungs-Pipeline mit Source + Target |
| `DataTrigger` | Manual / OnEvent / Scheduled |
| `FieldMapping` | Quell- → Zielfeld-Zuordnung mit Transform |
| `FieldTransform` | Direct / Template / Fixed |

## Trigger-Typen

| Typ | Beschreibung |
|-----|-------------|
| Manual | Nur manuell ausgelöst |
| OnEvent | Bei einem bestimmten Ereignis (z.B. `commit-pushed`) |
| Scheduled | Cron-Ausdruck (z.B. `0 8 * * *`) |

## Verzeichnis

| Pfad | Inhalt |
|------|--------|
| `src/controller.rs` | TaskController (Command Pattern) |
| `src/model.rs` | TaskPipeline, DataTrigger, FieldMapping |
| `src/view.rs` | TasksView, TaskDetailView, CreateTaskView (FsView) |
| `src/grpc.rs` | gRPC service |
| `src/rest.rs` | REST + OpenAPI |
| `src/cli.rs` | CLI |
| `src/keys.rs` | FTL-Schlüssel-Konstanten |
| `proto/tasks.proto` | gRPC Protokolldefinition |

## FTL-Strings

Alle UI-Texte in `fs-i18n/locales/{lang}/tasks.ftl`.

## Repo

`git@github.com:FreeSynergy/fs-tasks.git`
