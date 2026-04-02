# fs-bots

**Messenger-Bot-Manager für FreeSynergy**

fs-bots implementiert das **Strategy Pattern**: Jeder Bot-Typ (Broadcast, Gatekeeper, Digest, Monitor) hat eine eigene Strategie-Implementierung über `bot_strategy`.

## Architektur

```
BotController (Strategy Pattern via bot_strategy)
├── BotsView (FsView-Trait, view.rs)
├── GrpcBotsApp (gRPC: list/get/enable/disable/health)
├── REST + OpenAPI (axum + utoipa)
└── CLI (fs-bots)
```

## Bot-Typen

| Typ | Beschreibung |
|-----|-------------|
| Broadcast | Nachrichten an Räume/Gruppen versenden |
| Gatekeeper | Beitritte verifizieren und steuern |
| Digest | Tägliche Zusammenfassungen senden |
| Monitor | Dienste überwachen und benachrichtigen |

## Workspace-Crates

| Crate | Beschreibung |
|-------|-------------|
| `bots-ui` | Haupt-UI, gRPC, REST, CLI (Binary: `fs-bots`) |
| `bots` | Bot-Runtime (Matrix-Integration via fs-channel-matrix) |
| `fs-bot` | Bot-Framework (Command-Trait, Router, Rights) |
| `bot-db` | Persistenz-Layer (Räume, Subscriptions, Audit-Log) |
| `bot-broadcast` | Broadcast-Bot-Implementierung |
| `bot-calendar` | Kalender-Bot-Implementierung |
| `bot-control` | Control-Bot (Master-Messenger-Verbindung) |
| `bot-gatekeeper` | Gatekeeper-Bot-Implementierung |
| `bot-room-sync` | Raum-Synchronisations-Bot |

## Verzeichnis (bots-ui)

| Pfad | Inhalt |
|------|--------|
| `bots-ui/src/controller.rs` | BotController (Strategy Pattern) |
| `bots-ui/src/model.rs` | MessagingBot, BotKind, MessagingBotsConfig |
| `bots-ui/src/bot_strategy.rs` | BotStrategy-Trait |
| `bots-ui/src/view.rs` | BotsView (FsView) |
| `bots-ui/src/grpc.rs` | gRPC service |
| `bots-ui/src/rest.rs` | REST + OpenAPI |
| `bots-ui/src/cli.rs` | CLI |
| `bots-ui/src/keys.rs` | FTL-Schlüssel-Konstanten |
| `bots-ui/proto/bots.proto` | gRPC Protokolldefinition |

## FTL-Strings

Alle UI-Texte in `fs-i18n/locales/{lang}/bots.ftl`.

## Repo

`git@github.com:FreeSynergy/fs-bots.git`
