# Bots

[← Zurück zum Index](../INDEX.md) | [Bus](bus.md) | [Pakete](pakete.md) | [Rechte](rechte.md)

---

## Drei Programme, drei Aufgaben

| Programm | Aufgabe |
|---|---|
| **[Store](../programme/store/README.md)** | Bot-Module als Pakete finden, installieren, aktualisieren |
| **[Container App Manager](../programme/container_app/README.md)** | Bot konfigurieren (Tokens, Gruppen, Verbindungen, Plattform) |
| **[BotManager](../programme/botmanager/README.md)** | Bot **benutzen** (Broadcasts senden, Gatekeeper verwalten, Status sehen) |

Der BotManager ist ein **[Manager](manager.md)** — er lebt in `FreeSynergy.Managers` (`bots/`, Crate: `fsn-manager-bot`). Desktop bindet ihn als UI-Komponente ein (`app-botmanager`).

---

## Architektur: Ein Control-Bot mit Plugin-Modulen

Nicht viele separate Bots pro Gruppe, sondern **ein Control-Bot** der Module nachlädt:

```
Control-Bot (immer installiert, Admin-Rechte)
  ├── /broadcast Modul    ← Bus-Events → Messenger-Nachrichten
  ├── /gatekeeper Modul   ← Gruppen-Beitritt verifizieren
  ├── /tickets Modul      ← Ticket-System
  ├── /calendar Modul     ← Termine anzeigen
  └── weitere Module aus dem Store
```

Ein Bot in der Gruppe statt fünf. Jedes Modul ist ein `bot`-Paket im Store das nachgeladen werden kann.

---

## Control-Bot

Der Control-Bot ist der erste Bot der installiert wird. Nur Admins können ihn steuern.

**Funktionen:**
- Räume/Gruppen erstellen und verwalten
- Mitglieder einladen / entfernen
- Module nachladen (aus dem Store)
- Befehle von Admins empfangen
- Bus-Events empfangen und in Messenger-Gruppen weiterleiten
- Andere spezialisierte Bots einladen wenn nötig

### Control-Bot je Messenger

| Messenger | Bot-Typ | Crate | Volle Kontrolle? |
|---|---|---|---|
| Telegram | UserBot (MTProto) | `grammers` | Ja (Räume erstellen, Mitglieder verwalten) |
| Matrix | Normaler User mit Admin | `matrix-sdk` | Ja |
| Discord | Bot mit Admin-Permission | `serenity` + `poise` | Ja |
| Rocket.Chat | Admin-Integration (REST + WebSocket) | `reqwest` | Ja |
| Slack | App mit allen Scopes | `slack-morphism` | Fast alles |
| XMPP | Account + MUC-Admin | `xmpp-rs` | Ja |
| Mattermost | Admin-Integration | `reqwest` | Ja |

**Hinweis zu Telegram:** Für den Control-Bot wird ein **UserBot** benötigt, kein normaler Bot. UserBots laufen über MTProto mit einem echten Account und haben volle Kontrolle. Normale Bots (Bot-API via `teloxide`) sind für Gruppen-Verwaltung eingeschränkt.

---

## fsn-channel — Messenger-Unabhängigkeit {#fsn-channel}

Der Control-Bot schreibt **einmal**, `fsn-channel` übersetzt für jeden Messenger:

```rust
// Control-Bot-Code (messenger-unabhängig):
control_bot.create_room("projekt-koeln").await?;
control_bot.invite("projekt-koeln", user_id).await?;
control_bot.send("projekt-koeln", "Willkommen!").await?;

// fsn-channel übersetzt intern:
// Telegram:    grammers    → CreateChat + InviteToChat + SendMessage
// Matrix:      matrix-sdk  → create_room + invite_user + send_message
// Discord:     serenity    → create_channel + add_member + send_message
// Rocket.Chat: reqwest     → POST /api/v1/channels.create + invite + sendMessage
```

**Welcher Messenger aktiv ist, bestimmt das Inventory:** Der Control-Bot fragt nicht "Habe ich Telegram?" — er fragt das Inventory: "Welche Services haben die Rolle `chat`?" Wenn Telegram installiert ist, ist der Telegram-Channel aktiv. Wenn Matrix dazukommt, wird der Matrix-Channel ebenfalls aktiv. Der Bot-Code ändert sich nicht.

```rust
// fsn-channel liest das Inventory:
let channels = inventory.services_with_role("chat").await?;
// → [TelegramChannel, MatrixChannel] wenn beide installiert sind
// Neuer Messenger = neue Implementierung, kein Bot-Code-Change
```

`fsn-channel` lebt in `FreeSynergy.Lib` und wird vom Control-Bot als Dependency gezogen.

---

## Wichtiges Prinzip: Bots reden NIE direkt mit Services

Der Control-Bot redet **nicht** direkt mit Kanidm, Outline, Forgejo. Er geht **immer** über den Bus und die Bridges:

```
FALSCH:
  Bot → HTTP-Request → Outline API → Seite erstellen

RICHTIG:
  Bot → Bus-Event publizieren (Rolle: wiki, Topic: page.create)
    → Bus → Bridge (outline-wiki-bridge) → Outline API → Seite erstellen
```

Das ist das Grundprinzip: Rollen-basiert, nie direkt. Wenn Outline durch ein anderes Wiki ersetzt wird, funktioniert der Bot weiterhin ohne Änderung.

Gleiches gilt für den BotManager: Er redet nicht mit Messengern, sondern publiziert Bus-Events. Der Control-Bot empfängt sie und handelt.

```
BotManager → Bus-Event (bot.broadcast) → Control-Bot → fsn-channel → Messenger
```

---

## Rechte

Bot-Aktionen folgen der [Rechte-Kaskade](rechte.md). Wer was darf:

| Aktion | Benötigtes Recht |
|---|---|
| Bot-Status sehen | `read` |
| Broadcasts empfangen (Subscriptions) | `read` |
| Broadcasts senden | `execute` |
| Subscriptions verwalten (eigene Gruppen) | `write` |
| Gatekeeper: Anfragen sehen | `read` |
| Gatekeeper: Genehmigen / Ablehnen | `execute` |
| Module aktivieren / deaktivieren | `write` |
| Bot-Tokens ändern | `execute` (Admin) |

**Standard: Alles privat.** Kein Bot hat Zugriff auf fremde Projekte solange das Projekt nichts explizit freigibt. Wenn eine Föderation Broadcasts weiterleiten darf, dann nur weil das Projekt `execute`-Recht für die Bot-Rolle freigegeben hat — und Föderationen können dieses Recht nicht erhöhen, nur weitergeben oder einschränken.

---

## Bus-Events (Überblick)

| Topic | Richtung | Beschreibung |
|---|---|---|
| `bot.broadcast` | BotManager → Control-Bot | Nachricht in Gruppen senden |
| `bot.gatekeeper.approve` | BotManager → Control-Bot | Beitrittsanfrage genehmigen |
| `bot.gatekeeper.deny` | BotManager → Control-Bot | Beitrittsanfrage ablehnen |
| `bot.subscription.add` | BotManager → Control-Bot | Subscription hinzufügen |
| `bot.subscription.remove` | BotManager → Control-Bot | Subscription entfernen |
| `bot.status.request` | BotManager → Control-Bot | Status abfragen |
| `bot.status.response` | Control-Bot → BotManager | Status-Antwort |
| `chat.join_request` | Control-Bot → BotManager | Neuer Beitrittsantrag |
| `chat.join_approved` | BotManager → Control-Bot | Genehmigung ausführen |

---

## Kanal-Abonnements (Broadcast-Modul)

Das Broadcast-Modul verbindet den Message Bus mit Messenger-Gruppen:

```
User in Telegram-Gruppe schreibt: /subscribe git.commit
  → Control-Bot registriert Subscription im Bus:
      Rolle: git, Topic: git.commit → Ziel: Gruppe "projekt-koeln"
  → Ab jetzt: Jeder Commit erzeugt ein Bus-Event → Bot postet in die Gruppe

User schreibt: /subscribe wiki.page.created
  → Neue Wiki-Seiten werden in die Gruppe gepostet

User schreibt: /unsubscribe git.commit
  → Subscription entfernt
```

Jede Gruppe kann verschiedene Topics abonnieren. Benutzer konfigurieren das für ihre eigenen Gruppen (wenn sie die Rechte haben). Der BotManager bietet dafür eine grafische Oberfläche.

---

## Bot-Module als Store-Pakete

Bot-Module sind `bot`-Pakete im Store. Jedes Modul deklariert seine Commands, Triggers und benötigten Rollen:

```toml
[package]
id = "bot-broadcast"
name = "Broadcast Module"
type = "bot"
version = "1.2.0"
icon = "bot-broadcast"
tags = ["bot", "broadcast", "notification", "subscribe"]
description = "Subscribe to Bus-Events and receive notifications in Messenger groups"

[bot]
parent = "control-bot"
commands = [
    { name = "/subscribe",     params = ["topic"],     description = "Subscribe to events" },
    { name = "/unsubscribe",   params = ["topic"],     description = "Unsubscribe" },
    { name = "/subscriptions", description = "List active subscriptions" },
]
required_roles = []
triggers = ["*"]              # Kann auf alle Bus-Events reagieren
```

```toml
[package]
id = "bot-gatekeeper"
name = "Gatekeeper Module"
type = "bot"
version = "1.0.3"
icon = "bot-gatekeeper"
tags = ["bot", "gatekeeper", "iam", "verification"]
description = "Verify group join requests against IAM"

[bot]
parent = "control-bot"
commands = [
    { name = "/verify",  params = ["user_id"],    description = "Verify a user via IAM" },
    { name = "/approve", params = ["request_id"], description = "Approve join request" },
    { name = "/deny",    params = ["request_id"], description = "Deny join request" },
]
required_roles = [{ roles = ["iam"], mode = "ANY" }]
triggers = ["chat.join_request"]
```

Ein Modul mit `required_roles` ist nur **aktiv** wenn ein passender Service installiert ist. Wenn kein IAM-Service installiert ist, bleibt das Gatekeeper-Modul inaktiv — es wird aber nicht entfernt. Sobald IAM installiert wird, aktiviert es sich automatisch.

Jeder kann Bot-Module schreiben und in den Store stellen. Andere installieren sie und laden sie in ihren Control-Bot.

---

## Store-Verzeichnis

Bot-Pakete liegen im Store unter:

```
Store/
└── shared/
    └── bots/
        ├── bot-broadcast/
        │   ├── manifest.toml
        │   └── ...
        ├── bot-gatekeeper/
        │   ├── manifest.toml
        │   └── ...
        └── bot-tickets/
            ├── manifest.toml
            └── ...
```

---

Weiter: [BotManager](../programme/botmanager/README.md) | [Tasks](tasks.md) | [Bus](bus.md) | [Inventory](inventory.md)
