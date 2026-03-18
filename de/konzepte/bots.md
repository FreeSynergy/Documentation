# Bots

[← Zurück zum Index](../INDEX.md) | [Bus](bus.md) | [Desktop](../programme/desktop/README.md)

---

## Drei Orte für Bots

| Ort | Funktion |
|---|---|
| **Store** | Bot finden und installieren |
| **Conductor** | Bot konfigurieren (Tokens, Gruppen, Verbindungen) |
| **Bot Manager** (Desktop) | Bot BENUTZEN (Broadcast senden, Gatekeeper verwalten) |

## Bot-Definition

Ein Bot ist ein Paket im [Store](../programme/store/README.md):
```toml
[package]
id = "broadcast"
name = "Broadcast Bot"
type = "bot"
tags = ["broadcast", "telegram", "matrix", "notification"]
```

## Geplante Bots

**Broadcast Bot:** Nachricht eingeben → an alle konfigurierten Gruppen/Kanäle senden
**Gatekeeper Bot:** Telegram-Gruppen-Join → Verifikation via IAM → Approve/Deny
**Personal Bot:** LLM-gestützt, kann auf Nachrichten im Namen des Users antworten (später)

---

## Channels (fsn-channel)

Ein Bot soll auf **allen Messengern gleichzeitig** funktionieren — nicht ein Telegram-Bot und ein Matrix-Bot und ein Discord-Bot, sondern **ein Bot, viele Kanäle**. Das ist die Aufgabe von `fsn-channel`: eine einheitliche Abstraktion über alle Messenger-APIs.

```rust
// Der Bot schreibt EINMAL:
control_bot.create_room("projekt-koeln").await?;
control_bot.invite("projekt-koeln", user_id).await?;
control_bot.send("projekt-koeln", "Willkommen!").await?;

// fsn-channel übersetzt das intern für jeden Messenger:
// Telegram  → grammers:   CreateChat + InviteToChat + SendMessage
// Matrix    → matrix-sdk: create_room + invite_user + send_message
// Discord   → serenity:   create_channel + add_member + send_message
```

### Unterstützte Messenger

| Messenger | Bot-Typ | Möglichkeiten | Crate |
|---|---|---|---|
| **Telegram** | UserBot (MTProto) | Volle Kontrolle: Räume erstellen, Mitglieder verwalten | `grammers` |
| **Telegram** | Bot-API | Eingeschränkte Rechte (kein Räume-Erstellen) | `teloxide` |
| **Matrix** | Normaler User mit Access-Token | Volle Kontrolle (Bot = normaler User) | `matrix-sdk` |
| **Discord** | Bot mit Admin-Permission | Server verwalten, Kanäle, Mitglieder | `serenity` + `poise` |
| **Rocket.Chat** | Admin-Integration (REST + WebSocket) | Räume erstellen, Nachrichten, Mitglieder | `reqwest` |
| **Slack** | App mit OAuth-Scopes | Channels, Nachrichten (kein Server-Erstellen) | `slack-morphism` |
| **XMPP/Jabber** | Normaler XMPP-Account | MUC-Räume erstellen, einladen — alles möglich | `xmpp-rs` |
| **Mattermost** | Integration (REST-API) | Channels, Nachrichten, ähnlich wie Slack | `reqwest` |

**Hinweis zu Telegram:** Für den Control-Bot wird ein **UserBot** benötigt, kein normaler Bot. UserBots laufen über das MTProto-Protokoll mit einem Account und haben volle Kontrolle. Normale Bots (Bot-API) sind eingeschränkt.

### Besonderheiten je Messenger

**Telegram:** Zwei separate APIs. UserBot (MTProto via `grammers`) für alles was volle Kontrolle braucht. Bot-API (HTTP via `teloxide`) für einfache Bots ohne Raum-Verwaltung.

**Matrix:** Kein Unterschied zwischen Bot und UserBot — jeder Account kann alles. Einfachster Messenger für Bot-Integration.

**Discord:** Bots brauchen explizite Permissions beim Server-Owner. Mit Admin-Berechtigung volle Kontrolle.

**Slack/Mattermost:** Bots sind "Apps" mit OAuth-Scopes. Server (Workspaces) können nicht programmatisch erstellt werden — nur Channels innerhalb eines bestehenden Workspaces.

**XMPP:** Vollständig offen. Normaler Account reicht für alles. Kein zentraler Server notwendig (föderiert).

---

## Control-Bot vs. Worker-Bots

Das Konzept: **ein Control-Bot** mit maximalen Rechten, der bei Bedarf spezialisierte Worker-Bots startet.

```
Control-Bot (Chef)
  └── verwaltet Räume, Mitglieder, Permissions
  └── startet Worker-Bots bei Bedarf
        ├── Broadcast-Bot   (sendet Nachrichten)
        ├── Gatekeeper-Bot  (verifiziert neue Mitglieder)
        └── Personal-Bot    (antwortet im Namen eines Users)
```

| Messenger | Control-Bot-Typ | Volle Kontrolle? |
|---|---|---|
| Telegram | UserBot (MTProto) | Ja |
| Matrix | Normaler User mit Admin-Rechten | Ja |
| Discord | Bot mit Admin-Permission | Ja |
| Rocket.Chat | Admin-Integration | Ja |
| Slack | App mit allen Scopes | Fast alles |
| XMPP | Normaler Account + MUC-Admin | Ja |

---

Weiter: [Tasks](tasks.md) | [Bus](bus.md)
