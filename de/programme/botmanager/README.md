# BotManager — Bot-Steuerung & Messenger-Integration

> Lebt in **FreeSynergy.Managers** (`bots/`), Crate: `fsn-manager-bot`.

[← Zurück zum Index](../../INDEX.md) | [Manager](../../konzepte/manager.md) | [Bots](../../konzepte/bots.md) | [Desktop](../desktop/README.md)

---

## Was der BotManager macht

Der BotManager ist der Manager für Bots. Er ist der Kleber zwischen dem Store (welche Bot-Module verfügbar sind), dem Inventory (welche Bots installiert und konfiguriert sind) und dem Benutzer.

```
Store ←→ BotManager ←→ Desktop / UI
              ↕
           Bus → Control-Bot → fsn-channel → Messenger
```

Er ist **nicht** für die eigentliche Messenger-Kommunikation zuständig — das macht der Control-Bot via `fsn-channel`. Der BotManager ist die UI um Bots zu **benutzen** und zu **verwalten**.

---

## Einstieg: Konten verbinden (Control Bot)

Der erste Schritt ist immer, den **Control Bot** mit Messenger-Konten zu verbinden.
Im Bereich "Konten" fügt man je Messenger die Zugangsdaten hinzu:

| Messenger | Benötigte Daten |
|---|---|
| Telegram | Bot-Token, Telefonnummer (UserBot) |
| Matrix | Homeserver-URL, Access Token |
| Discord | Bot-Token |
| Rocket.Chat | Server-URL, Benutzername, Passwort |
| Mattermost | Server-URL, Bot-Token |
| Slack | Bot-Token |
| XMPP | JID (user@server.com), Passwort |

Sobald ein Konto konfiguriert ist, verbindet sich der Control Bot automatisch.
Alle anderen Bot-Module (Broadcast, Gatekeeper, usw.) laufen dann über den Control Bot.

---

## Eigenständigkeit

Der BotManager läuft als Crate in `FreeSynergy.Managers`. Desktop bindet ihn als Komponente ein (`app-botmanager`). Er hat kein eigenes Binary — er ist eine Bibliothek mit UI-Komponenten.

---

## Views

### Accounts

Übersicht aller konfigurierten Messenger-Konten. Hier werden Zugangsdaten für den Control Bot hinterlegt (siehe [Einstieg: Konten verbinden](#einstieg-konten-verbinden-control-bot)). Jedes Konto zeigt seinen Verbindungsstatus (verbunden / Fehler / nicht konfiguriert).

### Bot-Status

Übersicht aller installierten Bots mit Status (online / offline / error):

```
Control Bot (Telegram)    ● online   | 3 Module aktiv
Control Bot (Matrix)      ● online   | 2 Module aktiv
Control Bot (Discord)     ○ offline  | Token fehlt → Container App Manager
```

Klick auf einen Bot → Detailansicht mit aktiven Modulen, letzter Aktivität, Fehler-Log. "Token fehlt" verlinkt zum Container App Manager zur Konfiguration.

### Broadcast

Nachricht an Messenger-Gruppen senden ohne Telegram/Matrix/Discord zu öffnen:

```
Empfänger:  [ Alle Gruppen ▼ ] oder einzelne Gruppen auswählen
Messenger:  [ Alle ▼ ] oder Telegram / Matrix / Discord...
Nachricht:  [___________________________________________]
            [ Markdown ]  [ Datei anhängen ]  [ Senden ]
```

Der BotManager publiziert `bot.broadcast` als Bus-Event — der Control-Bot empfängt es und sendet in alle abonnierten Gruppen.

### Subscriptions

Verwaltet welche Messenger-Gruppe welche Bus-Events empfängt:

```
Gruppe "Projekt Köln" (Telegram)
  ✅ git.commit          → Commits aus Forgejo
  ✅ wiki.page.created   → Neue Seiten in Outline
  ☐  tasks.assigned     → Aufgaben-Zuweisung

[ + Subscription hinzufügen ]
```

### Gatekeeper

Verwaltung offener Beitrittsanfragen:

```
@max_muster möchte "Projekt Köln" beitreten
IAM-Status: ✅ Verifiziert (max.muster@helfa-koeln.de)
[ Genehmigen ]  [ Ablehnen ]

@unknown_user möchte "Team Süd" beitreten
IAM-Status: ❌ Nicht im System
[ Genehmigen ]  [ Ablehnen ]  [ Blockieren ]
```

Der BotManager publiziert `bot.gatekeeper.approve` / `.deny` — der Control-Bot führt die Aktion im Messenger aus.

### Bots

Installierte Bot-Module je Bot, aktiv/inaktiv (start/stop):

```
Control Bot (Telegram)
  📢 Broadcast Module    v1.2.0  ● aktiv
  🚪 Gatekeeper Module   v1.0.3  ● aktiv   (IAM installiert ✅)
  🎫 Tickets Module      v0.8.1  ○ inaktiv (kein Ticket-Service)

[ + Modul aus Store installieren ]
```

Inaktive Module haben `required_roles` die nicht erfüllt sind. Sobald der passende Service installiert wird, aktiviert sich das Modul automatisch.

---

## Rechte

Bot-Aktionen folgen dem [Rechte-System](../../konzepte/rechte.md):

| Aktion | Benötigtes Recht |
|---|---|
| Bot-Status sehen | `read` |
| Broadcasts empfangen | `read` |
| Broadcasts senden | `execute` |
| Subscriptions verwalten | `write` |
| Gatekeeper: Anfragen sehen | `read` |
| Gatekeeper: Genehmigen / Ablehnen | `execute` |
| Module aktivieren / deaktivieren | `write` |
| Bot-Tokens ändern | `execute` (Admin) |

---

## Bus-Integration

Der BotManager redet **nie direkt** mit Messengern — immer über den Bus:

| Event | Richtung | Beschreibung |
|---|---|---|
| `bot.broadcast` | BotManager → Control-Bot | Nachricht senden |
| `bot.gatekeeper.approve` | BotManager → Control-Bot | Beitritt genehmigen |
| `bot.gatekeeper.deny` | BotManager → Control-Bot | Beitritt ablehnen |
| `bot.subscription.add` | BotManager → Control-Bot | Subscription hinzufügen |
| `bot.subscription.remove` | BotManager → Control-Bot | Subscription entfernen |
| `bot.status.request` | BotManager → Control-Bot | Status abfragen |
| `bot.status.response` | Control-Bot → BotManager | Status-Antwort |
| `chat.join_request` | Control-Bot → BotManager | Neuer Beitrittsantrag |

---

## Bibliotheken

| Crate | Zweck |
|---|---|
| `dioxus` 0.7.x | UI-Komponenten |
| `fsn-bus` | Bus-Client (Events publizieren/empfangen) |
| `fsn-inventory` | Welche Bots/Module sind installiert? |
| `fsn-db` | SQLite (`fsn-botmanager.db`) |

**Nicht im BotManager:** `grammers`, `matrix-sdk`, `serenity` — diese gehören zum Control-Bot.

---

## Repo

https://github.com/FreeSynergy/Managers — Unterordner `bots/`

---

Weiter: [Bots Konzept](../../konzepte/bots.md) | [Manager](../../konzepte/manager.md) | [Desktop](../desktop/README.md) | [Bus](../../konzepte/bus.md)
